---
title: "Latency Assault"
date: 2021-01-28T22:22:16-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This middleware handler will generate a fixed or random delay in the request/response chain once it is enabled.

### Config

The following is the config file. 

latency-assault.yml

```
# Light Chaos Monkey Latency Assault Handler Configuration.

# Enable the handler if set to true so that it will be wired in the handler chain during the startup
enabled: true
# Bypass the current chaos monkey middleware handler so that attacks won't be triggered.
bypass: true
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 3
# Dynamic Latency range start in milliseconds. When start and end are equal, then fixed latency.
latencyRangeStart: 10000
# Dynamic latency range end in milliseconds
latencyRangeEnd: 10000

```

### Expectation

Depending on the configuration, a random latency will be injected during the normal request flow. If one request is picked to trigger the assault, a random or fixed delay will be added before returning the response to the client.

In the centralized log, you should find the following info level statement.

```
Chaos Monkey - I am sleeping for 1000 milliseconds!
```

The number of milliseconds is a variable in the statement.


In the metrics, you should see the response time for the endpoint has a spike periodically.

### Recovery

For each client request, a timeout should be configured, and it should retry another server instance in the cluster. 

If the timeout is not reached, then it is business as usual. 
