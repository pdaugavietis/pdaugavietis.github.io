---
layout: post
title: Setting Up Your SSH Config
subtitle: Tips and Tricks (and other evil thoughts)
tags: [ssh, linux]
bigimg: /img/path.jpg
published: true
---

We all use SSH to connect to remote (usually *nix) machines, and I've found a few tips and tricks (and a bonus VS Code trick at the end)...

## General Config Settings
So your configuration file lives at `~/.ssh/config` and here is where you can put various SSH client settings as you like.

    # apply this to ANY host connection
    Host *
        # for MacOS, use the KeyChain for passwords where possible
        IgnoreUnknown UseKeychain
        UseKeychain yes

        # instead of keeping a .ssh/known_hosts file with remote server
        # fingerprints, just ignore them
        UserKnownHostsFile /dev/null

        # ignore host key checking (against that known_hosts file that 
        # we threw away in the line above)
        StrictHostKeyChecking no

The last two settings there are likely not a good idea in all cases, but sometimes it's more convenient to ignore those and avoid potential conflicts (especially high-change environments like with containers).

## Connecting to AWS Instances with SSM Manager
An interesting way to connect to remote AWS nodes is using their instance ID, instead of their possibly changing, but difficultly named DNS, or dynamically assigned IP. With AWS SSM, you can call in, get the IP and connect to it; and with ProxyCommand support in SSH, have that all happen transparently.

    host i-*
        ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
        
        # set these if you know them ahead of time
        User ec2-user
        IdentityFile ~/workbox_only.pem

## Remote Connection Aliases
You can make aliases in your config files, to give a 'nice' name to machines that you frequently connect to.  In this case it's my remote working box:

    Host workbox
        HostName 1.2.3.4
        User ec2-user
        IdentityFile ~/workbox_only.pem
        UserKnownHostsFile /dev/null
        StrictHostKeyChecking no
        ForwardAgent true
        ForwardX11 yes
        
        # remember that bonus I mentioned...
        LocalForward 5901 1.2.3.4:5901

This lets you use `ssh workbox` to connect, and then sets up the other options for you.  

> I mentioned that bonus before... If you install the remote editing plugins to Visual Studio Code (VSC), and include that last `LocalForward` command in the configs, you can then open VSC in remote mode, and edit the files on the remote workstation, on your local computer. This is nice when you need to edit files, but aren't either comfortable with VIM, or have to edit more than one file at a time.