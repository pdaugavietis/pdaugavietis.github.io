---
layout: post
title: Docker Medley
subtitle: Docker, Docker Compose and Other Thoughts
tags: [docker, docker-compose]
bigimg: /img/Containers.jpg
# published: false
---

So with containers, we usually start with Docker and work our way out from there. Today we'll talk about Docker, Docker Registry, Docker Compose, Docker Services, Docker Machine, Docker Swarms. Then building on those tools and concepts and moving to AWS EC2, ECS and EKS (but that last one is Kubernetes, which is another whole thing...).

# Docker (Engine)
The Docker engine is a starting point that lets most systems use Docker images, to encapsulate the functionality built into them. Docker images are effectively compiled artifacts that contain a subset of an operating system (I usually stick with Linux, but Windows is making an appearance now for containers too). So the encapsulation aspect of containers is a core concept to grasp.

Another concept is the idea of maximizing the "density" of compute resources on computer hardware. This has been a theme in computing all the way back to mainframes and LPARs (Logical Partitions). Virtual machines start building this idea out, where multiple virtual machines (VMs) can exist on single 'bare metal' server. Now taken further, you can 'slice' VMs further down and have multiple containers running on each virtual machine.

You can share resources between mulitple containers as well, but orchestrating that seemed a little difficult, running one container at a time and trying to wire them together. Something better was needed.

# Docker Registry
A Docker registry is a collection of Docker images, that hold many images, each with many tags. This can be considered the 'staging' area for Docker, and similar to Java concepts of Maven repositories. Docker daemons have a local repository that remote images are 'pulled' into and then 'run' on the local daemon. This is also where each image's 'layers' are stored, and ideally shared between containers and tags, since each image is just a collection of 'layers' (organized by hashcode) that contain the different 'pieces' of the image.

# Docker Compose
Docker Compose is a YAML based framework that lets you define multiple containers all acting together, exposing ports, defining volumes and networks; encapsulating the idea of a 'service' that's made up of multiple containers. This also lays the groundwork for a few concepts that we'll cover a bit later on; services that are groups of containers, but operating under one umbrella for organization purposes; and gaining the ability to scale containers, for performance and stability sake, gaining benefits around high availability (HA) and disaster recovery (DR).

# Docker Machine
Docker machine was another utility building on Docker, but would allow you to configure remote instances of Docker in places like EC2, Azure and other public and private cloud providers. It would allow you with a single command line call, instantiate a VM, install and configure Docker, register that node locally, and configure the local Docker daemon to use the endpoint exposed; making it so that your local `docker` commands actually were running on the remote machine. This saved the extra hassle of SSH'ing to the remote machine and running commands that way, but introduced the attack vector of exposing the remote API URL.

# Docker Swarm
A 'swarm' was how Docker extended itself across multiple nodes, to allow for larger and more diverse configurations of containers, as well built-in HA/DR. This was the joining into clusters (swarms), a series of Docker engines running on multiple machines, together to form one large compute 'pool'. This allowed containers to run wherever the swarm decided based on performance algorithms, and when using Docker services, allowed for those containers to be running where and as needed (and to what desired scale, or instance count). The single point of entry to the service would be an externally facing URI, and the routing system that's built into swarm would figure out where the backing container lived, and route the request accordingly.

# Moving on...
So we're met the major components and players in the Docker realm, but this was really the spark that lit another revolution in IT. Docker as a company went from Open Source, to Commercial, which triggered a whole series of advancements and evolutions from the elements outlined above.

## Open Container Initiative (OCI)
The OCI is an effort under the auspices of the Linux Foundation to develop specifications and standards to support container solutions. The documentation of those standards and specifications meant that there could be other, compatible competitors to Docker, and allow for a more diverse ecosystem of options.

### Containerd
Directly from their website, Containerd is "An industry-standard container runtime with an emphasis on simplicity, robustness and portability". It is built in Go (language) and allows programmatic access to run containers easily in code. Containerd is designed to be embedded into a larger system, rather than being used directly by developers or end-users.

### Podman/Skopeo/Buildah
Podman (Pod Manager) is a binary, that is effectively a drop-in replacement for the Docker engine. It allows to run all the same commands and CLI parameters, but do so without needing a central daemon. Each container is executed on it's own thread, and more importantly for DevSecOps, in it's own security context. This means that when a container is running, it connects to the outside world as a specific system-level user, and doesn't need to connect to something else that might be running as root, which eliminates certiain attack vectors through escalation of privileges.

Skopeo and Buildah are sister projects that allow you to perform various operations on container images and image repositories (without root permissions, and without an active daemon); and create a working container, either from scratch or using an image as a starting point (with or without a Dockerfile definition) and other container-related activities that are usually requiring a Docker daemon to accomplish.

## Amazon Web Service (AWS) Elastic Container Service
AWS ECS can be considered an evolution of docker-machine, swarms and docker-compose. There are a few major moving pieces - ECS 'servers', which are EC2 instances (one or more) that are configured to run 'Services', which are made of 'Tasks' (and Task Definitions), which are collections of 'Containers'. The best part here is that you can use a `ecs-cli` very much like `docker-compose`, and publish your docker-compose.yml files as Task Definitions, or ideally as 'Services' to allow the cluster to balance compute across nodes as needed.

AWS also has a complete set of offerings around containers, including public and private image registries (Elastic Container Registry - ECR) as well as CodeCommit tools for SCM, build automation and pipelines to go from source to images, including binary storage.

## Kubernetes
And we come to Kubernetes (k8s). K8s is an open-source system for automating deployment, scaling, and management of containerized applications. This now takes containers and idea of Docker to the next level, and I don't want to get into that quite yet, but we will in subsequent posts. Suffice to say that this was a major shift and scale and ability around containers, but still using the core OCI images, just allowing them to now grow as needed, and taking away some of the limits.

And yes we'll also talk about Elastic Kubernetes Service (EKS) which is AWS's flavour of K8s.  But not yet.