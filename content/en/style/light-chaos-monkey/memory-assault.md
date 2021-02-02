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

memory-assault.yml

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

This handler should only be used with a Docker container with memory limitation. If you run with your desktop, it would eat your memory and render your desktop very slow. 
