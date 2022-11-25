---
layout: post
title: Quick and Dirty -- What is my IP?
subtitle: Tips 'n Tricks
tags: [dev, tips, tricks]
published: true
---

You need your external or egress IP?

    $ curl ifconfig.co

    1.2.3.4

You can do a little script like this (or bash/zsh function):

    #!/bin/bash

    IP=$(curl ifconfig.co)

    echo "My IP: ${IP}"
    echo "CIDR: ${IP}/32"

That's it - quick and dirty!