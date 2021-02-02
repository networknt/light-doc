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

This is a middleware handler to generate an AssaultException during the request handling. As this is a runtime exception, it will bubble up and be handled by the exception handler at the beginning of the request/response chain. For example, the following is a possible setup for the default chain.


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

Here is the config file for the handler.

exception-assault.yml

```
# Light Chaos Monkey Exception Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will passthrough.
enabled: false
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5

```

