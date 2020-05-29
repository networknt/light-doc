---
title: "Rate Limiting"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Although our framework can potentially handle millions of requests per second, some public-facing APIs might be a good idea to enable this handler to limit the concurrent requests to a certain level to avoid DDOS attacks.

As this handler will impact the overall performance a little bit, it is not configured as default in the [light-codegen](https://github.com/networknt/light-codegen). You must config the feature as true in your light-codegen config.json or update the config later in the deployment phase.

## Dependency

The following dependency needs to be added to pom.xml in your project to use the handler.

1.6.x release with JDK8
```
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>limit</artifactId>
                <version>${version.light-4j}</version>
            </dependency>
```

2.0.x release with JDK11

```
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>rate-limit</artifactId>
                <version>${version.light-4j}</version>
            </dependency>
```


## Service

As this middleware is not plugged in by default, we need to add it to the handler.yml in the config folder. As this rate-limiting handler needs to be failed fast, it needs to be put right after ExceptionHandler and MetricsHandler. The reason it is after MetricsHandler is to capture 513 error codes in InfluxDB and Grafana for monitoring on production.


```
#Traceability Put traceabilityId into response header from request header if it exists
com.networknt.traceability.TraceabilityHandler
#Rate Limiting
com.networknt.limit.LimitHandler
#Metrics In order to calculate response time accurately, this needs to be the second.
com.networknt.metrics.MetricsHandler
#Exception Global exception handler that needs to be called first.
com.networknt.exception.ExceptionHandler

```

## Config

Here is the configuration file for rate limiting.

```
# Rate Limit Handler Configuration

# If this handler is enabled or not
enabled: false

# Maximum concurrent requests allowed, if this number is exceeded, request will be queued.
concurrentRequest: 1000

# Overflow request queue size. -1 means there is no limit on the queue size
queueSize: -1

```

- Update enabled to true to enable it and false to disable it.
- concurrentRequest number of concurrent requests to be limited.
- queueSize -1 unlimited queue size, which might use a lot of memory. > 1 integer will limit the requests to be queued, and once the queue is full, 513 will be returned for new requests. 
