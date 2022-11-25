---
layout: post
title: Dive, Dive - We're Goin' In!
subtitle: Using Dive to analyze Docker images
tags: [devops, docker]
published: true
---

[Dive](https://github.com/wagoodman/dive) is a tool for exploring a docker image and analyzing the layer contents.  You can use it to look at what's in each image, and look to improve your layer re-use and optimize your images.

```
$ dive groovy
Image Source: docker://groovy
Fetching image... (this can take a while for large images)
...
```

![dive-groovy](/assets/img/screenshots/dive-groovy.png)

Here you can view each layer's contents in the right pane,  while navigating the layers in the left side.

You can also run the analysis to run in more a CI mode:

```
$ dive --ci groovy
  Using default CI config
Image Source: docker://groovy
Fetching image... (this can take a while for large images)
Analyzing image...
  efficiency: 98.9487 %
  wastedBytes: 3914519 bytes (3.9 MB)
  userWastedPercent: 1.8398 %
Inefficient Files:
Count  Wasted Space  File Path
    3        2.2 MB  /var/cache/debconf/templates.dat
    3        512 kB  /var/log/dpkg.log
    3        364 kB  /var/lib/dpkg/status
    2        322 kB  /var/log/lastlog
    2        276 kB  /var/lib/dpkg/status-old
    3         55 kB  /var/log/apt/history.log
    3         55 kB  /var/cache/debconf/config.dat
    2         35 kB  /var/log/faillog
    2         34 kB  /var/log/apt/term.log
    3         24 kB  /etc/ld.so.cache
    3         20 kB  /var/cache/ldconfig/aux-cache
    3         18 kB  /var/log/apt/eipp.log.xz
    2        7.2 kB  /var/log/alternatives.log
    3        5.1 kB  /var/lib/apt/extended_states
    2        1.9 kB  /etc/passwd
    2        1.0 kB  /etc/shadow
    2         907 B  /etc/group
    2         759 B  /etc/gshadow
    2         300 B  /var/lib/dpkg/diversions
    3           0 B  /var/lib/apt/lists
    3           0 B  /var/lib/dpkg/lock-frontend
    3           0 B  /var/lib/dpkg/triggers/Lock
    2           0 B  /var/lib/dpkg/triggers/Unincorp
    3           0 B  /var/cache/debconf/passwords.dat
    3           0 B  /var/lib/dpkg/updates
    3           0 B  /var/cache/apt/archives/partial
    3           0 B  /var/cache/apt/archives/lock
    5           0 B  /tmp
    2           0 B  /etc/.pwd.lock
    3           0 B  /var/lib/dpkg/lock
Results:
  PASS: highestUserWastedPercent
  SKIP: highestWastedBytes: rule disabled
  PASS: lowestEfficiency
Result:PASS [Total:3] [Passed:2] [Failed:0] [Warn:0] [Skipped:1]
```

This lets you integrate `dive` into your pipelines and value streams for testing and validation.