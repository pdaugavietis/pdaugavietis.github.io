---
layout: post
title: AWS EKS - EKS, ALB and DNS - Oh my!
subtitle: What else do we need INSIDE an EKS cluster?
tags: [aws, eks, kubernetes]
published: true
---

So we have an EKS cluster.  What now?  We want a more functional EKS cluster that's ready to accept deployments.  Awesome.  But what's that, and what should I put there, BEFORE I'm open for (container) workloads?  

> What follows is a slightly opinionated (my opinion) look at what a 'functional' cluster means.  And for clarity, you can run Kubernetes (k8s) workloads right away on a provisioned EKS cluster, but there many nice-to-haves from an operational point-of-view that I don't think are optional - like an Application Load Balancer (ALB) ingress controller, Cert Manager for SSL, External DNS contoller for custom DNS names and some basic log handling that will send logs to CloudWatch instead of having to SSH to machines, or `docker exec` into containers to look at things.

### Where we start
Let's assume that we start with the following eks-fargate.yaml file (see [this post](/2021-02-04-eks-clusters-from-yaml) for background on eksctl):

```eks-fargate.yml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: cluster-fg
  region: us-east-2

iam:
  withOIDC: true
  serviceAccounts:
  - metadata:
      name: k8s-admin
      labels: { aws-usage: "cluster-ops" }
    attachPolicyARNs:
    - arn:aws:iam::aws:policy/AdministratorAccess
  - metadata:
      name: aws-load-balancer-controller
      namespace: kube-system
      labels: { aws-usage: "cluster-ops" }
    attachPolicyARNs:
    # you will need to create this policy first
    - arn:aws:iam::695067481560:policy/AWSLoadBalancerControllerIAMPolicy
  - metadata:
      name: external-dns-controller
      namespace: kube-system
      labels: { aws-usage: "cluster-ops" }
    attachPolicyARNs:
    # you will need to create this policy first
    - arn:aws:iam::695067481560:policy/AllowExternalDNSUpdates
    - arn:aws:iam::aws:policy/AmazonEKSClusterPolicy 

fargateProfiles:
  - name: fp-default
    selectors:
      # All workloads in the "default" Kubernetes namespace will be
      # scheduled onto Fargate:
      - namespace: default
      # All workloads in the "kube-system" Kubernetes namespace will be
      # scheduled onto Fargate:
      - namespace: kube-system
```
Let's apply it:

```
❯ eksctl create cluster -f eks-fargate.yml

[ℹ]  eksctl version 0.36.2
[ℹ]  using region us-east-2
[ℹ]  setting availability zones to [us-east-2a us-east-2c us-east-2b]
[ℹ]  subnets for us-east-2a - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for us-east-2c - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for us-east-2b - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  using Kubernetes version 1.18
[ℹ]  creating EKS cluster "fargate-cluster" in "us-east-2" region with Fargate profile
...
[✔]  all EKS cluster resources for "fargate-cluster" have been created
[ℹ]  kubectl command should work with "/Users/pdaugavietis/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "fargate-cluster" in "us-east-2" region is ready
```

> Note that you can also add YAML content to enhance the cluster and avoid messages like this one:
> ```
> [ℹ]  CloudWatch logging will not be enabled for cluster "fargate-cluster" in "us-east-2"
> [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-2 --cluster=fargate-cluster'
> ```
> Please see [eksctl (under Usage)](https://eksctl.io/) for more options

So out of this we get:

- a cluster named "fargate-cluster" running in `us-east-2`
- an IAM OIDC provider, connected to our cluster
- a service account for k8s, with IAM permissions granted through OIDC
- a Fargate profile that will accept workloads from `kube-system` and `default` namespaces

If we check some kubectl commands:

```
❯ kgno

NAME                                                   STATUS   ROLES    AGE    VERSION
fargate-ip-192-168-122-83.us-east-2.compute.internal   Ready    <none>   94s    v1.18.8-eks-7c9bda
fargate-ip-192-168-156-61.us-east-2.compute.internal   Ready    <none>   100s   v1.18.8-eks-7c9bda

❯ kgns

NAME              STATUS   AGE
default           Active   7m29s
kube-node-lease   Active   7m30s
kube-public       Active   7m31s
kube-system       Active   7m31s
```
> I use macOS, with Oh-My-Zsh and PowerLevel10k, which comes with a whole stack of kubernetes (and other popular tools) shortcut aliases. Don't be alarmed if you see them here, it's just an aliased command to save on typing (like above where `kgns` is abbreviated from `kubectl get namespaces`).

Okay - looks like we're in business. Now one thing to note here, is with Fargate clusters, you have to specify the namespaces that will get scheduled into the Fargate nodes, and if the desired namespace isn't assigned to a Fargate profile - YOUR CONTAINERS WILL NOT START!  So right now, we can **ONLY** use the default namespace or the kube-system namespace for workloads that we want to schedule.

### What we still need
So we can schedule pods and services, get persistent volume claims and volumes - but to really be 'ready' for primetime we need:

- an Application Load Balancer (ALB) ingress controller
    - expose our k8s services over HTTP or HTTPS
    - configure in-line rules to redirect from HTTP to HTTPS automatically
- a Certificate Manager controller
    - used by ALB controller to handle SSL certificates from AWS Certificate Manager (ACM)
- an External DNS controller
    - used by ALB controller to provide custom DNS names configured in Route53

### Application Load Balancer controller (and cert-manager)

##### Verify and save that we have an OIDC IAM provider with the EKS cluster:

```
$ OIDC_PROVIDER=$(aws eks describe-cluster --name cluster-fg --region us-east-2 --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///"); echo $OIDC_PROVIDER
oidc.eks.us-east-2.amazonaws.com/id/XXXXXXC237E30B488F360C9CE4XXXXXX
```

Save the VPC ID that the cluster is using:

```
$ VPC_ID=$(aws eks describe-cluster --name cluster-fg --region us-east-2 --query "cluster.resourcesVpcConfig.vpcId" --output text); echo $VPC_ID
vpc-074fe16698XXXXXXX
```
##### Download the IAM policy for AWS LoadBalancers:

```
$ curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.1.0/docs/install/iam_policy.json
```

and create the IAM policy:

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

##### Create the IAM service account with that policy:

```
eksctl create iamserviceaccount \
    --cluster=<cluster-name> \
    --region=<region-code> \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
    --approve
```

##### Configure EKS Helm chart repository:

```
$ helm repo add eks https://aws.github.io/eks-charts

```

> If you don't have Helm installed, then [follow the instructions here](https://helm.sh/docs/intro/install/).

##### Install the Application Load Balancer helm chart:

```
$ helm install alb-ingress eks/aws-load-balancer-controller \
  --set installCRDs=true \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set serviceAccount.create=false \
  --set clusterName=<cluster-name> \
  --set region=<region> \
  --set vpcId=<cluster-vpc-id>
```

##### Verify its running

```
$ kga --all-namespaces | grep alb
default       pod/alb-ingress-aws-load-balancer-controller-6f4dd79d9b-jtdxd   1/1     Running   0          8m6s
default       deployment.apps/alb-ingress-aws-load-balancer-controller   1/1     1            1           8m7s
default       replicaset.apps/alb-ingress-aws-load-balancer-controller-6f4dd79d9b   1         1         1       8m8s
```

### External DNS controller

##### Create IAM policy file for external DNS:

```
cat > AllowExternalDNSUpdatesPolicy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
        "Effect": "Allow",
        "Action": [
            "route53:ChangeResourceRecordSets"
        ],
        "Resource": [
            "arn:aws:route53:::hostedzone/*"
        ]
        },
        {
        "Effect": "Allow",
        "Action": [
            "route53:ListHostedZones",
            "route53:ListResourceRecordSets"
        ],
        "Resource": [
            "*"
        ]
        }
    ]
}
EOF
```

##### Create the IAM policy:


```
aws iam create-policy \
    --policy-name AllowExternalDNSUpdatesPolicy \
    --policy-document file://iam-policy.json
```

##### Create the IAM service account for external-dns:


```
eksctl create iamserviceaccount \
    --cluster=<cluster-name> \
    --region=<region> \
    --namespace=kube-system \
    --name=external-dns-sa \
    --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AllowExternalDNSUpdatesPolicy \
    --attach-policy-arn=arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
    --approve
```

Get the role arn for the service account:

```
$ eksctl get iamserviceaccount \
  --cluster <cluster-name> \
  --region <region>

NAMESPACE	NAME				ROLE ARN
default		k8s-admin		arn:aws:iam::<account_id>:role/eksctl-eksctl-dev-addon-iamserviceaccount-de-Role1-XXXXXXXXXXXXX
```

##### Create the manifest for external-dns:

Update the role-arn using the value from previous step:

```
cat > external-dns.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
  namespace: kube-system
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns-controller
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: kube-system
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
      # If you're using kiam or kube2iam, specify the following annotation.
      # Otherwise, you may safely omit it.
      annotations:
        iam.amazonaws.com/role: arn:aws:iam::ACCOUNT-ID:role/IAM-SERVICE-ROLE-NAME
    spec:
      serviceAccountName: external-dns-controller
      containers:
      - name: external-dns
        image: k8s.gcr.io/external-dns/external-dns:v0.7.3
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=basedomain.com # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
        - --provider=aws
        - --policy=upsert-only # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
        - --aws-zone-type=public # only look at public hosted zones (valid values are public, private or no value for both)
        - --registry=txt
        - --txt-owner-id=my-hostedzone-identifier
      securityContext:
        fsGroup: 65534 # For ExternalDNS to be able to read Kubernetes and AWS token files
EOF
```

##### Update the `iam.amazonaws.com/role`, `--domain-filter` and `--txt-owner-id`; then apply the manifest:

```
kubectl apply -f external-dns.yaml
```

## Okay let's go...
So now we have all the moving parts in the cluster ready - let's deploy something.  I'll cover that in the next post...
