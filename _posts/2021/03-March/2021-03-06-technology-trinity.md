---
layout: post
title: The Trinity - People, Process, Tools
subtitle: Holy or not, all things come in threes...
tags: [devops]
published: true
---

In Information Technology (IT), and especially in DevOps, there is a common trinity of People, Processes, and Tools.  Those three sides can be summarized like this:

- Tools are used by People to follow a process.
- People follow a Process that is enforced by Tools.
- A Process is performed by People who are using Tools.

These three points of the triangle are important, and sadly sometimes overlooked, in DevOps; and need to considered to the same level of detail and importance.  If any one of these foci skews too far, or too little, the chances for success in DevOps implementations and day-to-day operations in a DevOps value stream become difficult, if not impossible.

## Tools
Tools are an important component to the entire value stream, but at the same time it's only one point of three.  Tools, however, is where a fair bit of the flashiness of DevOps lives, since the 'thing' is a tangible construct that can be touched, used, and looked at.  This is also where most of the "DevOps" labels are concentrated, like Continuous Integration (CI) and Continuous Delivery (CD).  You may have also heard before that DevOps isn't tools (or JUST tools), and that is exactly the message that you should process.  Tools are here to help enforce a process that makes sense for your teams, but should never dictate the process that you want to follow.  Tools are there to support solid processes like Infrastructure or Configuration-As-Code, with things like Terraform, CloudFormation, Ansible, Puppet; with CI processes like the ability to perform builds and tests atomically, and by commit, with things like Jenkins, Bitbucket Pipelines, Github Pipelines, GitLab Pipelines; and Observability (of systems and applications), with things like Prometheus, Grafana, New Relic and Datadog.

## Processes
Lets take a look at Processes.  This is a true cornerstone of DevOps, as the processes form a map, or guide, and provide direction to the value stream, telling us where we need to go, and how we should be getting there.  The processes need to be understood as well, or the provisioning of tools can grow without limits; costing the company dearly in dollars, but especially with time spent (or rather wasted) chasing white rabbits.  Processes also add an element of 'generality', where a process is 'less a set of rules, but more guidelines really' (arrgh), and with the deluge options out there for tools (and everyone's personal preferences), the real risk of 'snowflake'-enducing choices is really something to keep an eye out for.

DevOps is full of different processes, and at different 'levels' -- some are at the 10,000ft level, talking about Epics, Milestones and Summary Dashboard (executive level) while other can fall all the way down into the 'weeds', of unit testing, integration testing, branching strategies, version control branch naming conventions.  It's certainly enough to confuse the issues, and blur the lines between some levels, and send people chasing after those darned rabbits again.  And as mentioned above, you also need to notice if the tools that are in play for a particular aspect of the pipeline are affecting the process-level choices and forcing your hand one way, or another, you might just let that rabbit run away and find a new one.

## People
Oh boy - this is where the water really gets choppy.  When we start talking about people in the trinity, we have to also talk about culture.  Of the group, of the department, of the company at large, culture is such an important component of DevOps (success), that it really can't be overlooked.  And truly, there are so many landmines here to step on:

- hoarding of knowledge (intentional or not) is never a good thing
- having individuals in mind when problems come up (as the go-to-guy/gal) never ends well
- living in fear of making the smallest of mistakes, losing your job, ending up in a gutter... well you get it...

The idea of 'blameless' post-incident reviews, 'safe' retrospectives and generally a culture of open and honest dialog, without fear of repercussions is essential in building a highly functional, collaborative team of DevOps professionals.  We all need to learn from each other, cover each other off, and generally make the workplace (or even heaven-forbid, the world) a better place to be.

## Summary
DevOps is a three-sided triangle, that is weighted differently for everyone out there, but needs to be kept at least a triangle of concerns.  What comes out of this trinity, is a solid DevOps practice that helps everyone deliver code faster, with less errors, at a faster rate -- all the way to production!  And the word 'practise' here is the most important, as you need to do all three of these things all the time (dare I say continuously), since practice makes perfect!


