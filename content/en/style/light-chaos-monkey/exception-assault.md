---
title: "Exception Assault"
date: 2021-01-28T22:22:33-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This is a middleware handler to generate an AssaultException, a runtime exception, during the request handling. As this is a runtime exception, it will bubble up and be handled by the exception handler at the beginning of the request/response chain. For example, the following is a possible setup for the default chain.

```
chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - killapp
    - latency
    - memory
    - exchaos
    - specification
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

```

### Config

Here is the config file for the handler.

exception-assault.yml

```
# Light Chaos Monkey Exception Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will passthrough.
enabled: false
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5

```

### Expectation

The client should receives the following error. 

```
{"statusCode":500,"code":"ERR10010","message":"RUNTIME_EXCEPTION","description":"Unexpected runtime exception","severity":"ERROR"}
```

In the centralized log, the following info level log statement can be found. 

```
Chaos Monkey - I am throwing an AssaultException!
```

### Recovery

When the client receives this error, it means the particular server instance is not functioning. The client should try to look up another instance in the cluster and retry the same request if client-side discovery is used.

On the server-side, the error code "ERR10010" should be captured in the centralized log, and an alert should be sent out to the support team to start the investigation. 

