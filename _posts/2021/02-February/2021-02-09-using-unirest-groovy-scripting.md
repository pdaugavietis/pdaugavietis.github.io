---
layout: post
title: Feeling Groovy - UniREST
subtitle: How can I call REST services from Groovy?
tags: ['java', 'groovy']
published: true
---

I used to be a Java developer, so my go-to scripting language tends to be Groovy - that and professionally I am (was?) an Atlassian consultant, and wrote plugins and scripts in Groovy for that as well...

So integrating to other services is usually done with REST calls (GET, POST, PUT, etc.) and to do that in Groovy is sometime more confusing by using the built-in classes (URI, HTTPConnection, etc.) - but there is an easier way: [Unirest](http://kong.github.io/unirest-java/)!

## The Basics
To use external resources like this, you can use Grapes (which is based on Ant and similar to Maven, for Groovy projects) to download the runtime dependencies.

```
@Grapes([
        @Grab(group = 'com.konghq', module = 'unirest-java', version = '3.7.02', classifier = 'standalone'),
        @Grab(group = 'com.fasterxml.jackson.core', module = 'jackson-core', version='2.11.0')
])
```

You can then configure Unirest (using the Command pattern):

```
Unirest
    .config()
    .defaultBaseUrl("https://rest-service.somewhere.com")
```

## How do I use it?
Then you can use the Unirest object to perform the actual HTTP operations:

```
HttpResponse<JsonNode> response = Unirest.get("/rest/api/something")
        .basicAuth(username, password)
        .header("accept", "application/json")
        .asJson()
```

This will call the GET operation on the service, using BASIC authentication and get the response from the service as a JSON object.  We can turn that JSON object into a simple List of generic Objects, which will let you use simple Collection iterators and accessors:

```
ObjectMapper mapper = new ObjectMapper()
List<Object> maps = mapper.readValue(response.body.toString(), new TypeReference<List<Object>>() {});

maps.findAll { it.state == "OFFLINE"}.collect { it.nodeId }.each { nodeId ->
    Unirest.delete("/rest/api/2/cluster/node/${nodeId}")
            .basicAuth(username, password)
            .header("accept", "application/json")
            .asEmpty()
}
```

Here we now use the Collections.findAll closure to search the returned list of maps for all the maps in the list that have a state of "OFFLINE", and then the Collections.collect closure to select ONLY the "nodeId" key/values and then use that list as inputs for the DELETE operation.

## The Kit (and Kaboodle)
Below we have the script in its entirety, and it's used to clean out Jira Data Center nodes that are offline (since there isn't a direct way to do that in the GUI by default).

```
@Grapes([
        @Grab(group = 'com.konghq', module = 'unirest-java', version = '3.7.02', classifier = 'standalone'),
        @Grab(group = 'com.fasterxml.jackson.core', module = 'jackson-core', version='2.11.0')
])

def username = "user"
def password = "password"

Unirest.config().defaultBaseUrl("https://jira-dc-server")

HttpResponse<JsonNode> response = Unirest.get("/rest/api/2/cluster/nodes")
        .basicAuth(username, password)
        .header("accept", "application/json")
        .asJson()

ObjectMapper mapper = new ObjectMapper()
List<Object> map = mapper.readValue(response.body.toString(), new TypeReference<List<Object>>() {});

map.findAll { it.state == "OFFLINE"}.collect { it.nodeId }.each { nodeId ->
    Unirest.delete("/rest/api/2/cluster/node/${nodeId}")
            .basicAuth(username, password)
            .header("accept", "application/json")
            .asEmpty()
}

response = Unirest.get("/rest/api/2/cluster/nodes")
        .basicAuth(username, password)
        .header("accept", "application/json")
        .asJson()

println response.body
```