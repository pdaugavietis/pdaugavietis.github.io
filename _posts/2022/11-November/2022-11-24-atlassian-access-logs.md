---
layout: post
title: Atlassian Baselining and Troubleshooting
subtitle: A starting point for metrics and data
tags: [devops, observability]
published: false
date: "2022-11-24"
---

So we all know that Atlassian apps are Java-based (they usually run on a version of Apache Tomcat) and that there are many ways to solve the "I need data, to make the data-driven decisions" about heap size and configuration, garbage collection and general performance.  This is dancing around the issue of observability, both internal and external.

Let's start with external, since it's a little easier and less invasive in our stacks.

## External Observation Points

Out of the box we get an Apache-style access log in the Tomcat server's `server.xml` file.  We can format it, change it, tweak it, but generally speaking this is the easiest way to get a feel for how the server is running, without incurring possible overhead (we'll talk about that later, with the internal options).

The section we mean is something like this (pulled from `/opt/atlassian/confluence/conf` in the `atlassian/confluence-server` Docker container):

    <Valve className="org.apache.catalina.valves.AccessLogValve"
        requestAttributesEnabled="true"
        directory="logs"
        prefix="confluence_access"
        suffix=".log"
        rotatable="true"
        pattern="%h %{X-AUSERNAME}o %t &quot;%r&quot; %s %b %D %U %I &quot;%{User-Agent}i&quot;" />

Tomcat uses 'valves' to enable the access log functionality, and we can see that in this configuration, the logs will be prefixed with `confluence_access` in the `$SERVER_HOME/logs` directory.  A sample of the output is below:

    172.17.0.1 admin [23/Nov/2022:23:59:30 +0000] "GET /rest/troubleshooting/1.0/check/admin?_=1669247969899 HTTP/1.1" 200 57 138 /rest/troubleshooting/1.0/check/admin http-nio-8090-exec-8 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
    172.17.0.1 admin [23/Nov/2022:23:59:30 +0000] "POST /rest/wrm/2.0/resources HTTP/1.1" 200 279 93 /rest/wrm/2.0/resources http-nio-8090-exec-10 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
    172.17.0.1 admin [23/Nov/2022:23:59:30 +0000] "POST /rest/analytics/1.0/publish/bulk HTTP/1.1" 200 40 13 /rest/analytics/1.0/publish/bulk http-nio-8090-exec-4 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"

Since this is a regular log file, it's ripe to be picked up by either Splunk, or Elastic's stack of Filebeats, Logstash, ElasticSearch and Kibana (this is the one we'll talk about, but it's just as easy in Splunk - it just comes with a cost!).  What we want to do is build a parser for the access log, to allow clean ingestion into the Elastic stack and allow us to analyse the logs later, and build nifty dashboards, configure alerts...  but that's another article.  For now the numbers that we want to pay attention to are just after the `HTTP/x.x`, a set of 3 numbers.

                                                                                          __________
    172.17.0.1 admin [23/Nov/2022:23:59:30 +0000] "POST /rest/wrm/2.0/resources HTTP/1.1" 200 279 93 /rest/wrm/2.0/resources http-nio-8090-exec-10 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"

Those numbers are the HTTP response code, number of bytes sent back, and how long (in millis) the request took to process.  Once ingested into a tool like Elastic, you can keep track of that over time, and see which URI's might be taking longer than usual, and give you hints about where to look.

But what about my JVM?

## Garbage Collection Logs

Its a little tricker to get information about the JVM internal functions, without going internal (which we'll talk about in a bit).  The one thing to definitely do is setup the Java garbage collection (gc) logging with JVM arguments (usually in the `/bin/start-confluence.sh` scripts, under `START_CONFLUENCE_JAVA_OPTS` or similar).

#### Simplest form

    -XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc

This turns on GC logging to `stdout` and makes the output verbose.

#### More complete (pre Java 9)

    -XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc
    -XX:+PrintGCDetailsw
    -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps

#### Java 9 and newer

JDK 9 introduced a `unified logging option` for gc logs:

    -Xlog:gc=<level>:<output>:<options>

    # enable gc logs at the debug level, send to 'gc.txt' file, and 
    -Xlog:gc=debug:file=gc.txt:pid,time,uptime

Once you get a file to look at, you can pass it through a log analyzer (like [gceasy.io](https://gceasy.io)) and see how the JVM is behaving, and in some cases, tell you how you can possibly improve your configuration.

> Note that the gc logging is NOT asynchronous (from what I've gathered so far) - https://stackoverflow.com/a/27652035

## Diving deeper

That's about all you can do from external observation, without generating some sort of performance overhead on your system.  Some options from here (that do affect system performance, at least a little bit):

#### Synthetic Checks
These are timed requests against the system (create an item, update item, perform some operation) that exercise the application and see how it's performing.  This will inject a very slight amount of load on the system, but is likely one of lightest impacts you can have.  Another benefit of these is that they can be run from different places (public cloud makes this silly easy now), and you can measure system performance (and user experience) from multiple places around the globe.

#### JMX Data Scraping
Java provides Java Management Extensions (JMX) which is a set of 'mbeans' (Management MBeans -- in going with the 'Java' theme) that hold a bunch of information about the JVM and how its running.  With a few configurations you can enable and expose them to external scraping tools and you can capture information from inside the JVM without incurring very much load, since it's all built-in and observational as well (I would rate the impact even less than Synthetics above).  You can also create your own custom mbeans, and pull information from other providers like Tomcat, some JDBC drivers and other Java-based tools.  Once exposed, you can use various collectors, like Telegraf or JMX2Graphite, to send the metrics and data to a storage backend like Graphite or Influx, then use tools like Grafana to visualize those metrics in dashboards.

#### Application Performance Monitoring (APM)
Here we get a little more complex.  You can use tools like Elastic APM, New Relic and Datadog to 'instrument' your application, and gain a wealth of information about your application, but that comes as a little more overhead to your running application.  This is similar to 'profiling' but not quite as impactful, as profiling usually has a much larger overhead, and is really meant for dev stacks to be a very deep check against the systems.  APM tools tend to only 'sample' a set of requests for that deeper view of data, and tend to be quite 'light' in impact these days.  The amount of [data, information and wisdom](https://pdaugavietis.github.io/2022-09-02-dikw/) that you get here is immense, but very complete as well.

In a followup we'll talk a little more detail about some of the tools mentioned above, and how they can help both your development cycle, and production observational cycle.