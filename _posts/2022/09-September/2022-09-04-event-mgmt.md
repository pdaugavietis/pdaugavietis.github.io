---
layout: post
title: Events -> Alerts -> Incidents -> Problems -> Changes
subtitle: Another progression?
tags: [devops, observability]
published: true
---

# Overview
This is a white paper that talks about monitoring concepts and starts to layer in a selected set of tools, but really concentrates on concepts and processes.  We'll also then touch on elements of DevOps, SRE and ITIL in terms of events, incidents, problems and changes.
One of the fundamental challenges is getting to a state of 'golden' metrics and data, in other words, getting the wheat from the chaff.  You can easily get inundated with data in monitoring, logging and metrics if you are not careful. The desire is to move up the pyramid (illustrated below) to gain the business value, as you go from data, to information, to knowledge, and to wisdom.

# Monitoring and Checks
The idea of monitoring and checks is to have certain checks and balances applied to our applications, to ensure that they are running well, and without any user-impacting issues.  We also do NOT want to be notified of system issues by clients, but rather be pro-actively checking things before they get bad enough to have a negative business impact.
In the next section we'll talk about event management, but first we need system to generate those events, in real (or near-real) time, based on certain criteria that may or may not be met.  Keep in mind also that ALL things are events, but only SOME are incidents.  You may want to keep track of certain data points, but you don't need to wake up support personnel at 3a on a Saturday.

## Log Aggregation
In today's datacenters, and even more in the ephemeral cloud, finding the logs for a given error, on a given system can be challenging.  Throw in micro services that might see 10 to 20 collections of log files per node, the problem explodes.  You pretty much have to centralize your logs to a common system, that can then be searched as needed, and generate the events that are important and need attention, automatically.  A few of the tools that would help with this are:
- Splunk
- Sumologic
- Elastic (Beats, Logstash, Elasticsearch, Kibana)

Once you have the logs coming to once place, its straightforward to have a common approach to event generation (for instance, when the term "ERROR" pops up in the logs), and filters can then be applied and improved upon to catch those events that are important, but also ignore the ones that aren't.  Ideally that decision is made more on the event management side, which keeps the event generation nice and simple.

## Checks
These are going to be point-in-time checks of a desired state, with an event generated with the state that was detected.  Ideally you would also measure the time it took to execute the check, what the payload size was, etc., as well to gather more relevant data around the check itself. These checks tend to very low-level (is port 22 open on a given host, is the URL for an application returning a value), and it's also important to consider at what level of the IEEE standard you're running checks on.  Ideally you want to be gathering as much information on a single check as you can.  Consider a web application running on port 8080 - if I do a layer 4 TCP check for port 8080 being open and active we prove the following:
- Something is listening on port 8080.
- Something is responding on port 8080.
- The firewall configurations are allowing traffic on port 8080.

However if I do a check of HTTP (which is layer 7) based on a CURL command, and inspect the results for a piece of text, I've now proven all the following:
- Something is listening on port 8080.
- Something is responding on port 8080.
- The firewall configurations are allowing traffic on port 8080.
- The application server is functioning correctly based on the HTTP 200 return code.
- The application server is functioning correctly since the expected text is found.

So making the application-based check provides a level of information (instead of just data), and more relevant to the desired state of the application health.

### Non-Transactional
These are the simpler checks to perform, but might have a more limited scope or value.
- URL checks
- Port checks
- Process checks

### Transactional
There are richer, more relevant checks, but can also affect the system under monitor by injecting extra load and 'test' data, unless that's taken into account in the application design.
- Synthetic checks
- Read-only checks
- Destructive checks

### Business and Technical Checks
Business units are likely not going to care if one URL or another is accessible, but they are interested in a general or summary view of their application's health.  Most modern monitoring systems, and especially status page type dashboard solutions, have the concept of nesting services (Pagerduty for instance calls them 'business services' that wrap 'technical services'.

## Application Performance Monitoring
APM for short, is all about wrapping up a set of checks that might be run against a given application, and move up the DIKW pyramid (see here for more on that).

### Checks vs. Metrics
Checks and metrics are very similar things, and most checks should (or at least could) generate metrics.  Metrics would be the data that is then kept for a period of time to compare against for trending purposes, or to go 'back in time' to help troubleshoot a negative event for a given application.

### Time-Series Database
Usually when capturing metrics, you will store them in a TSDB (for short) to keep data points for a period of time.  This allows for historical comparisons by day, month, quarter, etc., to help identify possible variances.  There are machine learning algorithms provided by some companies as well to detect 'outliers' or 'deltas' to the normal trends, and those calculations can be as complex as needed (and supported by data resolution).
This is also where you likely don't have an unlimited amount of storage, so you consider things like data retention, and data resolution for your stored metrics.  For instance you might only want to keep data for 12h at a 1s resolution (one data point for each second), then aggregate the metrics to 5m/10m/15m/60m aggregations to save both space and calculation time.

# Event Management
Everything thing that happens in a system is an event - and in fact your application architecture might rely on that fact (for event-driven architectures).  Some are good, some are bad, but every application that you have is going to generate hundreds, if not thousands, of events every minute – possibly every second!  We have to put some level of automation around this (one of the core tenets of DevOps is Automation of all things), especially at scale in a cloud-centric world, those events will stream need to stream into a system that can automatically evaluate the event, escalate it if needed, remediate it if possible, but track it regardless.

### Events
As we mentioned earlier, everything that happens in a system, at a point-in-time, is an event.  There are many types of events that you may want to capture:
- Garbage collection
- User login and logout
- Bad password counts
- Bad URLs
The last 2 items might be normal operations, where the user entered the wrong password, or URL.  But there are also cases where those events need to be escalated into 'alerts' that need some level of remediation.

### Alerts
Alerts are the next level of events, where some event has occurred and needs some attention.  Alerts might self-correct (if based on a timeout threshold, that for some cosmic reason slowed down on that last ping), and if you get an alert that keeps firing, and resolving – that's called 'flapping' and can indicate that thresholds are a little too tight, or your event system doesn't understand the concept of flapping.  Most modern event systems will support the idea of consecutive checks passing or failing to indicate the actual state (3 consecutive passes and 5 consecutive failures to indicate pass or fail, respectively).   If you have too many alerts, then you may want to declare (and escalate to) an 'incident'.

### Notifications
One of the simple distinctions between events and incidents is notifications.  Something has happened that needs someone's attention, we need to notify that support group or individuals that there is an incident to respond to.  Some of the following systems are most common:
- Slack (or other peer-to-peer messaging system, like Teams or Mattermost)
- Pagerduty (or other notifications and on-call system, like OpsGenie)
- Jira Service Management (or other work tracking tool, like Jira Software or Monday.com)
- Email/SMS
Leveraging something like Pagerduty for notifications is excellent for organizations that might have many team members, and multiple teams to keep track of will benefit from the built-in on-call schedule management, and nested business and technical services groupings that the product offers

#### Teams and Users
Most organizations are split into teams, and team members.  These teams are used to abstract individual users from services for support and notification reasons, and most tools also support the idea of nesting teams.

### Incidents
What if the system that validates passwords is down, and causing those issues?  What if the microservices that drive your application are not functioning.  You now have an event that needs attention, so we escalate that event to an incident.  Incidents are the beginning of the support pipeline that may involve rebooting the server, updating application configuration files or even code changes to the underlying applications themselves.
If we have many related incidents, we can escalate that to a 'problem'.

## Dashboards and Status Pages
There are several types of dashboards and visualization tools that can be used here, including simplified status pages that will provide at-a-glance status information of business systems (usually at the green/yellow/red light level).  When you want to look at something deeper than this summary level, you will usually start using more customized and detailed dashboards and search terms in either the log aggregators or metrics systems, to show the more detailed views of the systems desired.  Tools here would include:
- Grafana
- Datadog dashboards
- StatusPage
It's important to keep in mind to not complicate dashboards with too many metrics or comparisons – if a higher level status is desired, then that's what should be modelled.  If more detail is needed (for certain audiences perhaps) then a more targeted dashboard is likely more beneficial than trying to have "one dashboard to rule them all".

# Problem Management
If there too many incidents (either related or not), a 'problem' should be declared.  This is now going to trigger a 'problem management' workflow that might include the following:
- establish a 'tiger team' to solve the problem as soon as possible
- establish focussed communications (Slack channel, Zoom room, etc.)
- send notifications to management that a problem has been declared
- assign a problem 'owner' who will act as the point-of-contact for any teams involved

An important concept here is that you are connecting many 'lower' level incidents together into a problem record, and once that's done, you're able to act on the problem record and automatically update comments and status of the child objects.  Once you've resolved the problem record, the system should automatically resolve the children.  However, in order to solve a problem, most likely we'll need to change something, and raise a 'change' record.

# Change Management
In a production environment, it's important to keep track of changes that occur, in only to know what's actually running, where it's running and what version it's currently using.  One thing to note is also that a change-control process SHOULD NOT be a blocking activity or be too onerous to deal with (yes, even in higher risk cases).  The change control process SHOULD enforce testing, and through proper testing you build trust in the system, and the likely hood of change success or failure.

The process should also support different levels of risk and impact - higher risk or impact changes would likely need more approvals and/or validations -- but never forget that places like Amazon and Facebook do hundreds if not thousands of production releases per day -- so it's definitely possible.  The trick here will be the degree of testing in the pipeline, which is also why it's so important to automate testing, and have as much as possible (or at least, how much you need) to build trust in the systems being changed.

## Change Advisory Board (CAB)
The dreaded CAB is usually a set of more senior staff that are given oversight and responsiblity for production systems.  Ideally you should have both development and operational representation here, as well as supporting services like security, compliance and others - to make sure that all aspects of changes are considered.