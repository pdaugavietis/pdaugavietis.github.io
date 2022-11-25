---
layout: post
title: Java Application Baselining and Troubleshooting
subtitle: Let's talk tools - Jolokia and JMX2Graphite
tags: [devops, observability]
published: true
---

Last time out we talked a bit about what generally to look at.  This time the rubber meets the road, and we'll talk specifically about getting JMX data out of the JVM and putting it somewhere to use in dashboards and alerts.

## Lay of the Land

So we're going to talk about:

- [Jolokia](https://jolokia.org/)
    - From their website - "Jolokia is a JMX-HTTP bridge giving an alternative to JSR-160 connectors. It is an agent based approach with support for many platforms. In addition to basic JMX operations it enhances JMX remoting with unique features like bulk requests and fine grained security policies."
    - What this means is that we can use Jolokia to expose the JVM JMX-based information over an HTTP connection, to allow tools like JMX2Graphite, Telegraf or other scrapers to collect and store it.  The nifty part of Jolokia is there are a few different ways of getting this to work, including `javaagents` (which is what we'll use).  You can also run Jolokia as a stand-alone jar file in the same JVM that your application is running on, and dynamically attach to an already running process to collect data - WITHOUT having to restart the application server at the time.  Keep in mind that doesn't persist the connections between restarts, so is of limited, but functional use.
- [JMX2Graphite](https://github.com/logzio/jmx2graphite)
    - From their website: "jmx2graphite is a one liner tool for polling JMX and writes into Graphite (every 30 seconds by default). You install & run it on every machine you want to poll its JMX."
    - What this means is that you can run JMX2Graphite with either Docker or natively as a Jar file, and it will read all the data it can find in the JVM and send it to a Graphite backend for storage.  From there you can use a tool like Grafana to visualize the data into dashboards and raise alerts as needed.

## Jolokia

You can either use it as a java agent:

     -javaagent:/path/to/jolokia-jvm-<version>-agent.jar

or attach to a currently running Java process:

    java -jar /path/to/jolokia-jvm-<version>-agent.jar --help

There is a fair bit of customization available, but by default it will start a HTTP service on `localhost:8778` to connect to.  For more configuration options and syntax, see their [reference manual](https://jolokia.org/reference/html/agents.html#agents-jvm).

## JMX2Graphite

There are a few options here too - you can run as a standalone Docker container (preferred):

    docker run -i -t -d --name jmx2graphite \
        -e "JOLOKIA_URL=http://172.1.1.2:11001/jolokia/" \
        -e "SERVICE_NAME=MyApp" \
        -e "GRAPHITE_HOST=graphite.foo.com" \
        -e "GRAPHITE_PROTOCOL=pickled" \
        -v /var/log/jmx2graphite:/var/log/jmx2graphite \
        --rm=true \
        logzio/jmx2graphite

or with a configuration file (see [sample](https://github.com/logzio/jmx2graphite/blob/main/application.conf)):

    docker run -d --name jmx2graphite \
        -v path/to/config/myConfig.conf:/application.conf \
        logzio/jmx2graphite

Running in a container is suggested as it provides the most flexibility in configuration topology, but you can also just run the command standalone, or even as a java agent (without the need for Jolokia, but the functionality isn't as complete).

## Graphite and Grafana

This far down the road is a little out-of-scope for this entry, but you can try running a Graphite stack in Docker containers:

    docker run -d\
        --name graphite\
        --restart=always\
        -p 80:80\
        -p 2003-2004:2003-2004\
        -p 2023-2024:2023-2024\
        -p 8125:8125/udp\
        -p 8126:8126\
        graphiteapp/graphite-statsd

This should start a Graphite stack with StatD running as well for ingestion.  You can go [here](https://github.com/graphite-project/docker-graphite-statsd) for more details.