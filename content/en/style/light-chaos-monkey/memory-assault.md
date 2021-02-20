---
title: "Memory Assault"
date: 2021-01-28T22:22:57-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

It is a middleware handler that will eat free memory once it is triggered based on the config file below.

### Config

memory-assault.yml

```
# Light Chaos Monkey Memory Assault Handler Configuration.

# Enable the handler if set to true so that it will be wired in the handler chain during the startup
enabled: true
# Bypass the current chaos monkey middleware handler so that attacks won't be triggered.
bypass: true
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

This handler should only be used with a Docker container with memory limitation. If you run with your desktop, it would eat your memory and render your desktop very slow. 


### Expectation

When a request triggers this attack, the request will have no response, and the middleware handler will try to eat the free memory until out of memory. 

If the server runs in a Docker container with limited memory allocated, the container will be shut down and restarted by the schedular. 

In the centralized log, the following info level log statement can be found. 

```
Chaos Monkey - I am eating free memory!
```

### Recovery

The current request will get time out or IOException depending on the timeout setup. And it should retry the same request with another server instance. 

All subsequent requests should go to another instance. 

The server instance should be restarted by the scheduler automatically.
