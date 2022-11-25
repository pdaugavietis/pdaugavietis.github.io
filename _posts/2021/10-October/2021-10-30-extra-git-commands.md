---
layout: post
title: Extra Git Commands
subtitle: Yeah yeah, commit and push right?
tags: [development, git]
published: true
---

Everybody knows Git right -- add, commit, push, pull...  Simple right?  Well there a whole bunch of other commands and functionality to use, so let's check some out...

## So much typing... or Aliases
Programmers are inherently lazy creatures - I speak from experience.  If you do it once - fine.  If you do it twice - script it. 

When you edit files, or add new ones, usually it's something like:

    <edit file>
    git add . (or use '-A' if you add a new file)
    git commit -am 'commit message'
    git push

That's too much work - let's create an alias:

    git config --global alias.ac '!git add -A && git commit -m'

Now our workflow is a little easier.

    <edit files>
    git ac 'message' 
    git push

Ahh - that's a bit better.

## Oops - didn't mean to do that...
How many times have you commited the wrong thing...  YES YOU HAVE!!!  ADMIT IT!

    git revert <sha>

## Wait what was I working on?
So how can I check the commit history, view the logs or the last few commits?

    git reflog
    git log --graph --decorate --oneline
    git log -S (search git log for file contents)

## I'll get right back to you...
Ever need to quickly save your work, switch to the CORRECT branch and apply those updates?

    git stash / stash pop (local only, no commits)

Keep in mind that this does NOT create a commit, or save your work ANYWHERE other than locally, on your machine, temporarily.

## Or lord - we deleted that branch on the server, why do I still see it?
Don't forget that Git is a DISTRIBUTED system, so the remote is the same structure as locally, but *STRUCTURALLY* might be different - deleting branches after pull requests might clean them from the central server, but what about your local clone?

    git branch -vv
    git remote update --prune
    
Well that's close - but let's really clean then up locally...

    git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -d

