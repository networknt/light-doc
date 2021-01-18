---
title: "Light AWS Lambda"
date: 2020-12-08T16:48:56-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 12
weight: 12
slug: ""
toc: false
draft: false
reviewed: true
---

In the light-platform, we are using the middleware handlers to address cross-cutting concerns for microservices in the request/response chain in the same service process or within a light-proxy/router at the network level. 

With more and more serverless Lambda functions deployed to the AWS cloud, we need to find a way to address the cross-cutting concerns in the same fashion as the light-4j so that the Lambda functions can be integrated with the light ecosystem for security, logging, tracing, auditing and metrics. 

There are two different options to implement cross-cutting concerns for Lambda functions and you should choose one or the other based on the cost and performance. 

* [Lambda-Proxy][] - using the light-proxy to address cross-cutting concerns for Lambda functions. 

* [Lambda-Framework][] - cross-cutting concerns with interceptors, extensions within Lambda functions.


[Lambda-Proxy]: /style/light-aws-lambda/lambda-proxy/
[Lambda-Framework]: /style/light-aws-lambda/lambda-framework/
