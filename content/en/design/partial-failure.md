---
title: "Handling Partial Failure"
date: 2018-02-13T20:50:25-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "design"
    weight: 20
weight: 20
aliases: []
toc: false
draft: false
reviewed: true
---


### Introduction

In a distributed system there is the ever-present risk of partial failure. Since clients and services are separate processes or even reside on different physical servers, a service might not be able to respond in a timely way to a client’s request. A service might be down because of a failure or for maintenance. Or the service might be overloaded and responding extremely slowly to requests. Also, as services are distributed across networks or even data centers, it increases the risk of partial failures especially if you have too many small services interacting with each other to form a big application.

### Cascade Failure

Let’s say you have an aggregate service that calls five other services to serve a SPA running on the browser. If one of the five services is unresponsive, the aggregate service might block indefinitely waiting for a response. Not only would that result in a poor user experience, but in many applications it would consume a valuable resource called thread. Eventually the server will run out of threads and become unresponsive. If this happens in a long chain with a lot of services, one failure will be cascaded to others and to bring down the entire application. In a microservices architecture, a slow service is more detrimental than a dead service that fails fast to the client.

The situation is particularly serious in services built on top of Java EE stack due to the blocking nature. If all services are built on top of an asynchronous http server with non-blocking IO, the risk is significantly reduced. However, the proper setup is still important to safe-guard your application.

### Solution

To prevent the problem described above, it is essential that you design your services to handle partial 
failures.

A good approach to follow is the one described by Netflix. The strategies for dealing with partial 
failures include:

- Network timeouts 

Never block indefinitely and always use timeouts when waiting for a response. Using timeouts ensures that resources are never tied up indefinitely. Unlike Java EE applications, the timeout for microservices is usually shorter. A general guideline is 1 or 2 seconds and all services will be designed as fail-fast.

- Limiting the number of outstanding requests 

Impose an upper bound on the number of outstanding requests that a client can have with a particular 
service. If the limit has been reached, it is probably pointless to make additional requests, and those 
attempts need to fail immediately.

- Circuit breaker pattern 

Track the number of successful and failed requests. If the error rate exceeds a configured threshold, 
trip the circuit breaker so that further attempts fail immediately. If a large number of requests are 
failing, that suggests the service is unavailable and that sending requests is pointless. After a 
timeout period, the client should try again and, if successful, close the circuit breaker.

- Provide fallbacks 

Perform fallback logic when a request fails. For example, return cached data or a default value such as an empty set of objects. This is very useful for rendering Web pages in a single page application. We would rather have a small component empty on a page instead of waiting for that page forever.

- Client retry

When partial failures happen to update services in a chain, the data consistency between services is unknown and it will require the client to retry the request in order to ensure that consistency between multiple services are maintained. If we are doing it, all services in the chain must be idempotent so that the same request only updates the database exactly once. The request can only be started from the original client not in the middle of the service chain. For details, please refer to [idempotency][]. 


[idempotency]: /design/idempotency/
