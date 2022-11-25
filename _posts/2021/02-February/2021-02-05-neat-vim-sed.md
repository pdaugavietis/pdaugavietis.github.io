---
layout: post
title: "TIL: Sed (in VIM) search and replace (with confirmation)"
tags: [linux]
# bigimg: /img/path.jpg
published: true
---

Today I was editing a file on a remote linux machine, and wanted to do a search of text, and replace with confirmations, since I didn't want to replace all instances.  That's pretty simple:

    :s/search-text/replace-text/g

but I wanted confirmations of each instance found, if I wanted to replace it or not. So it's a minor difference:

    :%s/search-text/replace-text/gc

This will search for the text, but ask you to make sure you want to replace it.  On my instance (with VIM installed) the options where a simple 'y' or 'n', but in my readings today saw that sometimes (maybe with just vi, and not vim) it's 'c' to confirm the replacement.