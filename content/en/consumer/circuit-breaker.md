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
---

Circuit breaker can be used through Http2Client::getRequestService method, it uses a request wrapped by java.util.concurrent.CompletableFuture producer, in other words, whenever execute CircuitBreaker::call, it create a new request and execute it.

There is a specifc configuration on client.yml, e.g.:

```yml
request:
  errorThreshold: 5
  timeout: 3000
  resetTimeout: 600000
```

* errorThreshold: amount of timeout errors before change state for open (default value: 5).
* timeout: amount of time in milliseconds for wait for response of a request (default value: 3000).
* resetTimeout: amount of time in milliseconds for keep state open before change for half open (default value: 600000).

whether is not provided a value for any parameter, is adoted default value.