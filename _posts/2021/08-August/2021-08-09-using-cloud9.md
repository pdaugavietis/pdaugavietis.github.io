---
layout: post
title: Using Cloud9 for Development
subtitle: AWS Development
tags: [dev, aws]
published: true
---

From AWS:

    AWS Cloud9 allows you to write, run, and debug your code with just a browser. With AWS Cloud9, you have immediate access to a rich code editor, integrated debugger, and built-in terminal with preconfigured AWS CLI. You can get started in minutes and no longer have to spend the time to install local applications or configure your development machine

# Setup Cloud9
Navigate to Cloud9, select "Create Environment".

![code9](/assets/img/screenshots/code9-1.png)

![code9](/assets/img/screenshots/code9-2.png)

The important things to consider here are instance size (if you're going to running docker containers locally, then you might want a bigger VM), and the VPC/Subnet placement of your VM.  I won't go into where and why you should put your VM in a particular VPC or subnet, since it varies from person to person.

Once it's ready to go you'll see something like this:

![code9](/assets/img/screenshots/code9-ide.png)

Down the left side you'll see the folder structure on your VM, on the right are the editor and terminal windows to interact with the files.  Since this is just an Amazon based EC2 instance (in the background), you can install any utilities you need (other than GUIs, obviously) and they'll persist between uses until you destroy the Cloud9 instance.

Cloud9 also comes with AWS CLI installed, and will function in the security context of the creating user -- it's going to use your permissions.  If you end up requiring other permissions than those already granted to you, you'll have to talk to your administrator(s) to adjust IAM to your requirements.

    $ aws sts get-caller-identity
    {
        "Account": "XXXXXXXXXXXX", 
        "UserId": "XXXXXXXXXXXXXXXXXXXXX:user-id", 
        "Arn": "arn:aws:sts::XXXXXXXXXXXX:assumed-role/role-name/user-id"
    }

In following posts we'll start looking at other AWS Developer Tools, like CodeCommit and CodeBuild!