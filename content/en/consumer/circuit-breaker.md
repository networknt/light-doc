---
title: "Circuit Breaker"
date: 2019-04-25T14:58:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The circuit breaker can be used through Http2Client::getRequestService method, it uses a request wrapped by java.util.concurrent.CompletableFuture producer, in other words, whenever to execute CircuitBreaker::call, it creates a new request and executes it.

There is a specifc configuration on client.yml, e.g.:

```yml
request:
  errorThreshold: 5
  timeout: 3000
  resetTimeout: 600000
```

* errorThreshold: the number of timeout errors before change state for open (default value: 5).
* timeout: the amount of time in milliseconds for wait for the response of a request (default value: 3000).
* resetTimeout: the amount of time in milliseconds to keep state open before the change for half open (default value: 600000).

Whether it is not provided a value for any parameter, is adopted default value.

