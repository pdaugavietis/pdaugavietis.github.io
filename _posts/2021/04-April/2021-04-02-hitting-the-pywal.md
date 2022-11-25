---
layout: post
title: I hit a PyWal
subtitle: Dammit, that hurt...
tags: [osx, shell]
published: true
---

So playing with themes and colors on the CLI is fun, and when you run across an app like `pywal` that inspects a picture, and builds a theme around it, it sounds even better.

Until you bork the configs and your macbook's ...

- background goes pitch black and you can't change it
- menubar goes oddly translucent and you can't change it
- do a PRAM reset (several actually) and nothing changes

Then this happy thought:

- HOW MUCH of your weekend is now devoted to re-imaging your laptop :(

Then you spend hours searching around for the issue, until you find this:

```zsh
rm "./Library/Application Support/Dock/desktoppicture.db"
```

And you just saved yourself hours of restoring things.
