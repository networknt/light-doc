---
title: "Rate Limit"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Although our framework can potentially handle millions of requests per second, some public-facing APIs might be a good idea to enable this handler to limit the concurrent requests to a certain level to avoid DDoS attacks. 

When the http-sidecar, light-router or light-proxy is used inside a corporate network for a slow backend API, this handler can be used to throttle the requests to protect the backend API from being overwhelmed. 


As this handler will impact the overall performance a little bit, it is not configured as default in the [light-codegen](https://github.com/networknt/light-codegen). You must config the feature as true in your light-codegen config.yaml/config.json or update the handler.yml config later in the deployment phase.

### Dependency

To use the handler, the following dependency needs to be added to pom.xml in your project.

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


### handler.yml

As this middleware is not plugged in by default, we need to add it to the handler.yml in the config folder. As this rate-limiting handler needs to be failed fast, it needs to be put right after ExceptionHandler and MetricsHandler. The reason it is after MetricsHandler is to capture 503 or 429 error codes in InfluxDB and Grafana for monitoring on production.


```
handlers:
  # Rate limit middleware to prevent DDoS attacks externally or throttle requests internally
  - com.networknt.limit.LimitHandler@limit
.
.
.
.
.
.

chains:
  default:
    - exception
    - metrics
    - limit
    - traceability
    - correlation

```

### Config

Here is the configuration file for rate limiting.

```yaml
# Rate Limit Handler Configuration

# If this handler is enabled or not. It is disabled by default as this handle might be in
# most http-sidecar, light-proxy and light-router instances. However, it should only be used
# internally to throttle request for a slow backend service or externally for DDoS attacks.
enabled: ${limit.enabled:false}
# Maximum concurrent requests allowed at a given time. If this number is exceeded, new requests
# will be queued. It must be greater than 1 and should be set based on your use case. Be aware
# it is concurrent requests not the number of request per second. Light-4j server can handle
# millions of request per seconds. With 2 concurrent requests, the petstore API can still handle
# hundreds or even thousands requests per second.
concurrentRequest: ${limit.concurrentRequest:2}
# Overflow request queue size. -1 means there is no limit on the queue size and this should
# only be used in the corporate network for throttling. For Internet facing service, set it
# to a small value to prevent DDoS attacks. New requests will be dropped with 503 response
# code returned if the queue is full.
queueSize: ${limit.queueSize:-1}
# If the rate limit is exposed to the Internet to prevent DDoS attacks, it will return 503
# error code to trick the DDoS client/tool to stop the attacks as it considers the server
# is down. However, if the rate limit is used internally to throttle the client requests to
# protect a slow backend API, it will return 429 error code to indicate too many requests
# for the client to wait a grace period to resent the request. By default, 503 is returned.
errorCode: ${limit.errorCode:503}

```

- Update enabled to true to enable it and false to disable it.
- concurrentRequest number of concurrent requests to be limited.
- queueSize -1 unlimited queue size, which might use a lot of memory. > 1 integer will limit the requests to be queued, and once the queue is full, 503 will be returned for new requests by default. 
- errorCode can be changed to 429 if it is used internal to throttle the concurrent request to backend APIs. 
