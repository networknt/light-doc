---
title: "chaosmonkey post endpoint"
date: 2021-02-02T15:23:57-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

It is an endpoint that is enabled by default. It allows the user to update the configuration for the Chaos Monkey assault handlers during the runtime.

The same config file chaos-monkey.yml is shared with the [chaosmonkey get endpoint][].


```
# Light Chaos Monkey API handlers Configuration.

# Enable the handlers if set to true to allow user to get or post configurations for the assault handlers.
enabled: true
```

In most cases, we should disable all the assault middleware handlers and use this API to update the configuration to enable a specific handler for testing. 

Here is the path defined in the handler.yml

```
  - path: '/chaosmonkey/{assault}'
    method: 'post'
    exec:
      - security
      - body
      - chaospost

```

To access this endpoint, you need to provide the handler package as the path parameter. The following package names are valid.

* com.networknt.chaos.LatencyAssaultHandler
* com.networknt.chaos.ExceptionAssaultHandler
* com.networknt.chaos.KillappAssaultHandler
* com.networknt.chaos.MemoryAssaultHandler

If you are not sure of the package name, you can access the [chaosmonkey get endpoint][] to get all the configurations for each handler. 

As body parser handler is used in chain, we need to have the Content Type set as application/json in the header of the request. 

Here is a real request. 

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

For the real request example, please visit the [chaos monkey tutorial][]. 

[chaosmonkey get endpoint]: /style/light-chaos-monkey/getchaosmonkey/
[chaos monkey tutorial]: /tutorial/chaos-monkey/petstore/
