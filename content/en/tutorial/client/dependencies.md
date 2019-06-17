---
title: Client Dependencies
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: ""
weight: 10
sections_weight: 10
draft: false
toc: true
reviewed: true
---

If you are using the [Http2Client][] for the service to service communication, you need to include all client related modules into your pox.xml file. You can find these modules from [light-consumer-4j][] which is a convenient fat client that packages all consumer side modules together. 

If you are working with an original client, you'd better use the light-consumer-4j instead of including all modules individually. light-consumer-4j also implement retry and circuit breaker to further guarantee the connectivity to the APIs/services. 

This dependency will include the submodules required to support client consumption of an API like service discovery, load balancing, oauth2 integration, etc.

## Maven

```xml
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>light-consumer-4j</artifactId>
    <version>${version.light-4j}</version>
</dependency>
```


[Http2Client]: /concern/client/
[light-consumer-4j]: https://github.com/networknt/light-consumer-4j

