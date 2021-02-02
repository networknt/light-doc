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

This middleware handler will generate a fixed or random delay in the request/response chain. 

The following is the config file. 

latency-assault.yml

```
# Light Chaos Monkey Latency Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will pass through.
enabled: false
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 3
# Dynamic Latency range start in milliseconds. When start and end are equal, then fixed latency.
latencyRangeStart: 10000
# Dynamic latency range end in milliseconds
latencyRangeEnd: 10000

```
