---
layout: post
title: Docker Machine
subtitle: Docker, but running on another machine... somewhere else...
tags: [docker, docker-machine]
bigimg: /img/Containers.jpg
published: true
gh-repo: docker/machine
gh-badge: [star, fork, follow]
---

From: https://docs.docker.com/machine/

> Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. You can use Machine to create Docker hosts on your local Mac or Windows box, on your company network, in your data center, or on cloud providers like Azure, AWS, or DigitalOcean.

# Installation
This is for MacOS.  For others, consult the Github repo above, or exercise that Google-fu.

    $ curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine
    $ chmod +x /usr/local/bin/docker-machine

# Usage
(from https://docs.docker.com/machine/examples/aws/)

1. Optional, create `~/.aws/credentials` file with appropriate AWS keys to allow access to AWS APIs.
1. Run `docker-machine` with `amazonec2` driver and options to configure a new Docker machine in AWS.

        docker-machine create --driver amazonec2 \
            --amazonec2-open-port 8080 \
            --amazonec2-region ca-central-1 \
            --amazonec2-tags app,docker,env,dev \
            aws-docker

1. Check that `docker-machine` captured the instance (and verify in AWS console).
        
        $ docker-machine ls

        Creating CA: /Users/xxxxxxx/.docker/machine/certs/ca.pem
        Creating client certificate: /Users/xxxxxxx/.docker/machine/certs/cert.pem
        Running pre-create checks...
        Creating machine...
        (aws-docker) Launching instance...
        Waiting for machine to be running, this may take a few minutes...
        Detecting operating system of created instance...
        Waiting for SSH to be available...
        Detecting the provisioner...
        Provisioning with ubuntu(systemd)...
        Installing Docker...
        Copying certs to the local machine directory...
        Copying certs to the remote machine...
        Setting Docker configuration on the remote daemon...
        Checking connection to Docker...
        Docker is up and running!
        To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env aws-docker

        $ docker-machine ls

        NAME         ACTIVE   DRIVER      STATE     URL                       SWARM   DOCKER     ERRORS
        aws-docker   -        amazonec2   Running   tcp://1.2.3.4:2376                v20.10.2

![ec2-console](/assets/img/machine/ec2.png)

## Change Docker Engine Context
To use the new remote Docker machine:

    # view the co-ordinates of remote daemon
    $ docker-machine env aws-docker

    # apply those settings to current shell
    $ eval $(docker-machine env aws-docker)

    # unset to point back to localhost daemon
    $ docker-machine env -u

Once pointing to the remote daemon, any `docker` commands run locally will actually be executed on the remote side transparently to the user. This is especially useful to folks like me (on the wrong side of a slower internet connection), where downloading images (or building them) takes extra time and bandwidth.

Using this approach, makes Docker development and usage a whole lot nicer and easier.

