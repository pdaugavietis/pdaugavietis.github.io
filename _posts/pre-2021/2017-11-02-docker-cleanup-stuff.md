---
layout: post
title: How to clean up docker artifacts
# image: /img/DSC_2623.png
tags: [docker]
# published: false
---

Docker generates lots of fluff and needs a little bit of cleanup from time to time...

[Original credit here](https://gist.github.com/bastman/5b57ddb3c11942094f8d0a97d461b430)
## Cleanup volumes
```
$ docker volume rm $(docker volume ls -qf dangling=true)
$ docker volume ls -qf dangling=true | xargs -r docker volume rm
```

## Cleanup networks
```
$ docker network ls
$ docker network ls | grep "bridge"
$ docker network rm $(docker network ls | grep "bridge" | awk '/ / { print $1 }')
```

## Cleanup images
```
$ docker images
$ docker rmi $(docker images --filter "dangling=true" -q --no-trunc)

$ docker images | grep "none"
$ docker rmi $(docker images | grep "none" | awk '/ / { print $3 }')
```

## Cleanup containers
```
$ docker ps
$ docker ps -a
$ docker rm $(docker ps -qa --no-trunc --filter "status=exited")
```
