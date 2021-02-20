---
title: "chaosmonkey get endpoint"
date: 2021-02-02T15:23:39-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

It is an endpoint that is enabled by default. It allows the user to retrieve the configurations for the Chaos Monkey assault handlers during the runtime.

The same config file chaos-monkey.yml is shared with the [chaosmonkey post endpoint][].


```
# Light Chaos Monkey API handlers Configuration.

# Enable the handlers if set to true to allow user to get or post configurations for the assault handlers.
enabled: true
```

Here is the path defined in the handler.yml

```
  - path: '/chaosmonkey'
    method: 'get'
    exec:
      - security
      - chaosget

```

The following command from the terminal can be used to access the API.

```
curl -k https://localhost:8443/chaosmonkey
```

The result should be something like this.

```
{"com.networknt.chaos.LatencyAssaultHandler":{"enabled":true,"bypass":true,"level":1,"latencyRangeStart":1000,"latencyRangeEnd":60000},"com.networknt.chaos.ExceptionAssaultHandler":{"enabled":true,"bypass":true,"level":5},"com.networknt.chaos.KillappAssaultHandler":{"enabled":true,"bypass":true,"level":5},"com.networknt.chaos.MemoryAssaultHandler":{"enabled":true,"bypass":true,"level":5,"memoryMillisecondsHoldFilledMemory":90000,"memoryMillisecondsWaitNextIncrease":1000,"memoryFillIncrementFraction":0.15,"memoryFillTargetFraction":0.25}}

```

For the real request example, please visit the [chaos monkey tutorial][]. 

[chaosmonkey post endpoint]: /style/light-chaos-monkey/postchaosmonkey/
[chaos monkey tutorial]: /tutorial/chaos-monkey/petstore/

