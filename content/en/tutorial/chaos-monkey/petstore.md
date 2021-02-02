---
title: "Petstore Chaos Monkey"
date: 2021-02-01T22:53:27-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

To show users how to use the light-chaos-monkey handlers and APIs, we will copy the petstore project and make some changes to the configuration files. 

### Copy petstore

In the light-example-4j repository, we have a rest/openapi folder that contains a petstore project generated from the petstore OpenAPI 3.0 [specification](https://github.com/networknt/model-config/tree/master/rest/openapi/petstore/1.0.0). 

Copy the petstore folder to petstore-chaos-monkey, and we are going to update the project in the petstore-chaos-monkey folder.

### pom.xml

In the pom.xml, add the following dependencies. They are the middleware handlers and APIs from the light-chaos-monkey.

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>latency-assault</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>memory-assault</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>exception-assault</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>killapp-assault</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>chaos-monkey</artifactId>
            <version>${version.light-4j}</version>
        </dependency>


```

### handler.yml

We need to update the handler.yml to add these middleware handlers and API endpoints. 


```
handlers:
  # Chaos Monkey middleware handlers and API handler
  - com.networknt.chaos.KillappAssaultHandler@killapp
  - com.networknt.chaos.LatencyAssaultHandler@latency
  - com.networknt.chaos.MemoryAssaultHandler@memory
  - com.networknt.chaos.ExceptionAssaultHandler@exchaos
  - com.networknt.chaos.ChaosMonkeyGetHandler@chaosget
  - com.networknt.chaos.ChaosMonkeyPostHandler@chaospost

chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    # Chaos Monkey Middleware Handlers
    - killapp
    - latency
    - memory
    - exchaos
    - specification
    - security
    - body
    - audit
    - sanitizer
    - validator

  # Chaos Monkey API endpoints
  - path: '/chaosmonkey'
    method: 'get'
    exec:
      - security
      - chaosget

  - path: '/chaosmonkey/{assault}'
    method: 'post'
    exec:
      - security
      - body
      - chaospost

```

### chaos-monkey.yml

We need to add this config file to enable the Chaos Monkey APIs so that we can dynamically update the configuration for the assault handlers. 

```
# Light Chaos Monkey Latency Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will pass through.
enabled: true
```

### logback.xml

Let's update the logback.xml to enable info level for pacakge com.networknt and output to the stdout.

```
    <logger name="com.networknt" level="info" additivity="false">
        <appender-ref ref="stdout"/>
    </logger>

```

### latency-assault.yml

To simulate the latency assault, add the following config file to the config folder.

```
# Light Chaos Monkey Latency Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will pass through.
enabled: true
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 3
# Dynamic Latency range start in milliseconds. When start and end are equal, then fixed latency.
latencyRangeStart: 10000
# Dynamic latency range end in milliseconds
latencyRangeEnd: 10000
```

We set the level to 3 so that it is easy to have the attack encountered. The start and end of the latency range are fixed as 10 seconds. You can set up between 1 to 10 seconds by changing the latencyRangeStart to 1000. 

Now, let's send a request to the /v1/pets endpoint from the terminal.

```
curl -k https://localhost:8443/v1/pets
```

When you send several requests, one of the requests will have to wait for 10 seconds to respond. You can set a breakpoint in the LatencyAssaultHandler to capture the exact request that thread sleep is called.

Once we complete the test, we need to change the latency-assault.yml to `enable: false` so that we can focus on the next assault handler. 

### exception-assault.yml

We can use this handler to simulate runtime exceptions within the normal requests. Add the following config file to the config folder.

```
# Light Chaos Monkey Excpetion Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will passthrough.
enabled: true
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5
```

Restart the server and send the same request several times. On average, there would be 1 in 5 requests that you will receive the following response. 

```
curl -k https://localhost:8443/v1/pets
```

Exception result. 

```
{"statusCode":500,"code":"ERR10010","message":"RUNTIME_EXCEPTION","description":"Unexpected runtime exception","severity":"ERROR"}
```

Once we complete the test, we need to change the exception-assault.yml to `enable: false` so that we can focus on the next assault handler.

### killapp-assault.yml


To simulate the server crash during the normal request flow, we can set up the killapp-assault.yml to enable the KillappAssaultHandler.


```
# Light Chaos Monkey Kill App Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will pass through.
enabled: true
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5


```

Restart the server and send the request with curl as follows. 

```
curl -k https://localhost:8443/v1/pets
```

After several requests, you will see the server is shut down, and the following error returned.

```
curl: (52) Empty reply from server
```

### memory-assault.yml

Please only try the memory-assault with the Docker container with limited memory configured. It takes time to eat all the free memory on my desktop with big memory and render the desktop very slow. 

To enable it, add the following config file. 

```
# Light Chaos Monkey Memory Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will passthrough.
enabled: false
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5
# Duration to assault memory when requested fill amount is reached in ms.
# min=1500, max=Integer.MAX_VALUE
memoryMillisecondsHoldFilledMemory: 90000
# Time in ms between increases of memory usage.
# min=100,max=30000
memoryMillisecondsWaitNextIncrease: 1000
# Fraction of one individual memory increase iteration. 1.0 equals 100 %.
# min=0.01, max = 1.0
memoryFillIncrementFraction: 0.15
# Final fraction of used memory by assault. 0.95 equals 95 %.
# min=0.01, max = 0.95
memoryFillTargetFraction: 0.25
```

Send the same request repeatedly, and we will encounter the Out of Memory issue. 

### get chaosmonkey

The /chaosmonkey get endpoint can be used to get the configuration for all assault handlers. It is protected with a security handler. Normally, this is called from the light-controller from the light-portal with the portal JWT token. 

Send the following request from the terminal. 

```
curl -k https://localhost:8443/chaosmonkey
```

And the result should be something like the following.

```
{"com.networknt.chaos.LatencyAssaultHandler":{"enabled":false,"level":3,"latencyRangeStart":10000,"latencyRangeEnd":10000},"com.networknt.chaos.ExceptionAssaultHandler":{"enabled":false,"level":5},"com.networknt.chaos.KillappAssaultHandler":{"enabled":false,"level":5},"com.networknt.chaos.MemoryAssaultHandler":{"enabled":false,"level":5,"memoryMillisecondsHoldFilledMemory":90000,"memoryMillisecondsWaitNextIncrease":1000,"memoryFillIncrementFraction":0.15,"memoryFillTargetFraction":0.25}}
```

### post chaosmonkey

To update the configuation for a particular assault handler, we need to use the package name of the handler as path parameter. 

Send the following request from the terminal.

```
curl --location --request POST 'https://localhost:8443/chaosmonkey/com.networknt.chaos.LatencyAssaultHandler' \
--header 'Content-Type: application/json' \
--data-raw ' {
    "enabled": true,
    "level": 5,
    "latencyRangeStart": 1000,
    "latencyRangeEnd": 60000
}
'
```

The response will be the updated config for this handler. 


```
{
    "enabled": true,
    "level": 5,
    "latencyRangeStart": 1000,
    "latencyRangeEnd": 60000
}
```

The request will enable the latency-assault handler and set up the range from 1 second to 60 seconds. 

Please note that you must use the application/json as the content type in the header. 

### Summary

This tutorial is only the starting point to show you how to use the Chaos Monkey assault handlers in a microservice. For real applications, there would be many microservices to form a complicated invocation chain between them. With your automation tool in the pipeline, you can dynamically simulate different attacks on different nodes to test if the system can recover from the attacks. For example, if one service instance is down, the consumer service will retry with another provider instance. 

When we have time, we are going to integrate the Chaos Monkey handlers into the service discovery example with four different APIs chained together. Also, we are going to integrate the light-controller into the discovery example so that users can manipulate the attacks from the controller UI.

