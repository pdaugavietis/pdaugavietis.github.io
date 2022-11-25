---
layout: post
title: Using AWS CodeCommit for Git Repositories
subtitle: AWS Development
tags: [dev, aws]
published: false
---

AWS offers a whole slew of developer-centric tooling now, not just compute and database; but all the supporting cast that's needed for building CI and CD pipelines.  This includes:

    - CodeCommit - Git based repositories
    - CodeBuild - Automated build platform
    - CodeArtifact - Build artifact storage
    - CodeDeploy - Automated deployments
    - CodePipeline - Orchestration for all the tools

AWS also offers a cloud-based IDE called Cloud9, which if you haven't seen, you should checkout.  When you create it, the identity it assumes is your current role and permissions, but comes with a bunch of tools and features out-of-the-box to make life easier, and we'll use it here

Now let's talk a bit about the first one - CodeCommit!

# Setting up a repository
First, login to AWS or create a new account.  Then navigate to CodeCommit.
