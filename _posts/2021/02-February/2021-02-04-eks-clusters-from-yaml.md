---
layout: post
title: Building EKS Clusters with YAML
subtitle: Yet another way of building clusters...
tags: [aws, eks, kubernetes]
bigimg: /img/Containers.jpg
published: true
---

Building a Kubernetes cluster isn't too complicated, but at the same time - why OVER-complicate the process. You need somewhere to run your container workloads on AWS, you're beyond just firing up an EC2 instance and running Docker; and equally running an Elastic Container (ECS) solution is also not quite what you're after. You want the shiny (and slightly buzzword-y) Kubernetes (K8S).  [AWS EKS](https://docs.aws.amazon.com/eks/) lets you create clusters through the console, which is fine - but so 2010 ;P  We're all about the configuration and infrastructure as code right?  CloudFormation, Terraform, Ansible...  oh yeah that the stuff.  But it also comes with it's own set of perils and required baseline knowledge (if you want a hand with that, let me know - it's what I do during daylight hours :)

[eksctl](https://eksctl.io/) is a simple CLI tool for creating clusters on EKS - Amazon's new managed Kubernetes service for EC2. It is written in Go, uses CloudFormation, was created by Weaveworks, that let's you define an entire cluster for EKS in a single YAML file.  The types of clusters and EKS configurations are a little beyond the scope of this post, but I might try and cover that later as well.  You can look here for more detailed information: [AWS EKS](https://aws.amazon.com/eks).

## Installation
Follow the instructions [here to install](https://eksctl.io/introduction/#installation) on macOS, Linux or Windows.

## Create a cluster
The simplest type of cluster to use, is a Fargate based one to delegate the management of the compute nodes to AWS.  That YAML looks like this:

```bash
eksctl create cluster \
    --name my-cluster \
    --region us-west-2 \
    --fargate
```
eksctl then takes this command and generates a bunch of CloudFormation stacks and runs them.  Once the output finishes, you should see something like:

```
...
[✓]  EKS cluster "my-cluster" in "us-west-2" region is ready
```
And your cluster is ready to accept pods!

## Verify cluster
Now if you try and interact with the cluster, you should see something returned:

```
$ kubectl get service

NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
svc/kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   1m
```

## Something a little more complex
Here is an example that I use regularly to setup a base EKS cluster for various tasks:

```
# An example of ClusterConfig object using Managed Nodes
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksctl-dev
  region: ca-central-1

iam:
  withOIDC: true
  serviceAccounts:
  - metadata:
      name: k8s-admin
      labels: { aws-usage: "cluster-ops" }
    attachPolicyARNs:
    - arn:aws:iam::aws:policy/AdministratorAccess

managedNodeGroups:
  - name: base-nodegroup
    spot: true
    minSize: 3
    desiredCapacity: 3
    maxSize: 3
    availabilityZones: ["ca-central-1a", "ca-central-1b"]
    volumeSize: 20
    labels: {role: worker}
    tags:
      nodegroup-role: kube-system-worker
    iam:
      withAddonPolicies:
        albIngress: true
        externalDNS: true
        certManager: true
        ebs: true

cloudWatch:
  clusterLogging:
    enableTypes: ["all"]
```
Here we start doing a lot more with AWS and the infrastructure side, including:

- choosing the region our cluster will live
- setting up IAM components
    - creating an OIDC server connected to our IAM Settings
    - creating a new service account, and attaching IAM policies
- using managedNodeGroups
    - setting up 'spot' instances (to save $$)
    - choosing which AZs we want to have compute nodes for this cluster (DR/HA)
    - setting up 3 worker nodes, and labelling them as such
    - configuring those worker nodes with IAM roles for ALB ingress controllers, externalDNS, certificateManager and EBS
- setting up CloudWatch logs for all our nodes and containers
    
Once that cluster is up and running we can deploy pods, services and ingresses.  There are different types of ingresses, and I prefer to use the ALB ingresses where possible (we'll cover those in later posts) to expose the K8S services to the outside world (well, outside the cluster).  We can use ExternalDNS (in this case interfaced to Route53) to provide custom vanity names to either K8S services (which can be LoadBalancers, that end up as ELBs) or ingresses.  Certificate manager allows our ingress controllers to talk to Amazon Certificate Manager (ACM) and deal with our SSL certificate requirements as well.

I'll cover in more detail what all these pieces mean, and how to tweak them in a followup post.

