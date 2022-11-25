---
layout: post
title: Adding Maven Wrapper
subtitle: The thing I can never remember...
tags: [development, maven]
published: true
---

Simple one - to add Maven Wrapper to existing project:

    mvn -N io.takari:maven:wrapper

To specify Maven version:

    mvn -N io.takari:maven:wrapper -Dmaven=3.5.2

This sets up a project specific (and packaged) Maven binary, so you don't need Maven installed locally.
