---
layout: post
title: AWS EKS - Deploying Apps
subtitle: Let's DO something with this cluster!
tags: [aws, eks, kubernetes]
published: true
---

So in our [previous post](/2021-02-05-eks-bits-and-pieces), we configured an EKS cluster, got an ALB and external DNS integrations working on it.  Now we can deploy apps.

## Deploy what?  How?
Kubernetes is a container orchestration tool. This means that you tell it what you want to run (which images, and their configuration - called manifests), and k8s will figure out where in the cluster (if sufficient resources exist) to run the actual containers.

> The difference between images and containers is an image is the blueprint that an executing daemon uses to provide a running container to the user. You could consider this a similar 'thing', just at different stages of the lifecycle of an artifact (and the fact that you can build any number of containers, from a single image).

## What's a manifest?
Generally speaking, a manifest is just a YAML file with a specific structure:

```
# api version
apiVersion: v1
# kubernetes object type
kind: Service
# object metadata
metadata:
  name: jira-dc-svc
  namespace: atlassian
# object specifications
spec:
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  selector:
    app: jira-dc
```

- `apiVersion` identifies the type and version of the object
- `kind` identifies the type of object (from the API) that we're describing
- `metadata` is where the object name, namespace, labels and other descriptors live for this object
- `spec` is where we now describe the object `kind` and provide the specifications required to create the object
    - this will vary by object kind

## Do I need a manifest?
Actually you don't.  You can use `kubectl run` to start a container:

```
$ kubectl run podinfo --image stefanprodan/podinfo
pod/podinfo created
```

Here is a comparable manifest to do the same thing:

```
apiVersion: v1
kind: Pod
metadata:
spec:
  containers:
  - image: stefanprodan/podinfo
    name: podinfo
```

You can now use either `kubectl describe pod` or `kubectl get pod` to see if the pod is running:

```
$ kdp podinfo
Name:         podinfo
Namespace:    default
Priority:     0
Node:         controller/172.18.0.2
...
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m4s  default-scheduler  Successfully assigned default/podinfo to controller
  Normal  Pulling    2m3s  kubelet            Pulling image "stefanprodan/podinfo"
  Normal  Pulled     2m2s  kubelet            Successfully pulled image "stefanprodan/podinfo"
  Normal  Created    2m2s  kubelet            Created container podinfo
  Normal  Started    2m2s  kubelet            Started container podinfo
```

```
$ kgp podinfo
NAME      READY   STATUS    RESTARTS   AGE
podinfo   1/1     Running   0          3m5s
```

## A way to test pods
There is a way to connect to active pods by forwarding a local port to the container in question:

```
$ kubectl port-forward podinfo 9898:9898
Forwarding from 127.0.0.1:9898 -> 9898
Forwarding from [::1]:9898 -> 9898
...
```

You can then connect to http://localhost:9898 and test the running pod.


## How do I connect the running pod from other pods?
The pod is base runtime object, but the pod is not accessible directly from the outside world - it's running on it's own internal network (internal to the Kubernetes cluster).  You need a service to access it, of either type `LoadBalancer` or `NodePort` (`ClusterIP` is the default type, but that's still internal to the k8s cluster - this is also a internal 'load balancer' to abstract the pods (that can scale for performance if needed) from the outside world.

You would 'expose' pods with a service:

```
$ kubectl expose pod podinfo --port 9898 --name podinfo-svc
service/podinfo-svc exposed

$ kgs
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
podinfo-svc   ClusterIP   10.108.119.192   <none>        9898/TCP   6s
```

To access the pods from OUTSIDE the k8s cluster we want to expose the service as either a `NodePort` or `LoadBalancer`:

```
$ kubectl expose service podinfo-svc --port=9898 --target-port=9898 --name=podinfo-ext-svc --type=NodePort
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
podinfo-ext-svc   NodePort    10.111.184.187   <none>        9898:32339/TCP   2s
podinfo-svc       ClusterIP   10.108.119.192   <none>        9898/TCP         15m
```

## How do I connect the running pod from outside the cluster
Now if you connect to http://10.111.184.187:32339 you should see the podinfo.  If you choose `LoadBalancer` you would see something under EXTERNAL-IP, the the target port (`32339` above) would NOT be auto-generated.  If you're using EKS, the `LoadBalancer` type is provided by Elastic Load Balancing (ELB).

If you want to use something a little fancier, you would look to leverage an ingress (can only be created by applying a manifest):

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: podinfo-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "podinfo-svc"
              servicePort: 9898
```

Note that `ingress`es are an extension to Kubernetes and not native objects, so their implementation is not guaranteed by ANY Kubernetes provider, and is usually supplied by the specific implementation that you want to use (`nginx` in the example above).  I'm not going to into much more detail here about configuring ingresses, but we'll talk about that in later posts.