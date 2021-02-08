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

In the light-platform, we are using the middleware handlers to address cross-cutting concerns for microservices in the request/response chain in the same service process or within a light-proxy/light-router at the network level. 

With more and more serverless Lambda functions deployed to the AWS cloud, we need to find a way to address the cross-cutting concerns in the same fashion as the light-4j so that the Lambda functions can be integrated with the light ecosystem for security, logging, tracing, auditing and metrics. 

There are two different options to implement cross-cutting concerns for Lambda functions, and you should choose one or the other based on your non-functional requirements. 

* [Lambda-Proxy][] - using the light-proxy to address cross-cutting concerns for Lambda functions. 

* [Lambda-Framework][] - cross-cutting concerns with interceptors, extensions within Lambda functions and custom authorizer on the AWS API Gateway for JWT verification.

Most users want to focus on the business logic implementation for the Lambda functions instead of boilerplate code. Regardless of which approach to use, the light-codegen can scaffold the Lambda function project based on the OpenAPI specification. 

* [Lambda Proxy Generation][] - scaffolding a project that leverages the light-proxy
* [Lambda Framework Generation][] - scaffolding a project that leverages the Lambda framework

When building Lambda functions in Java, the JVM runtime is prone to [code start latency](https://hackernoon.com/im-afraid-you-re-thinking-about-aws-lambda-cold-starts-all-wrong-7d907f278a4f). What if we create a native executable binary for Linux from our Java code. This is what GraalVM tooling enables us to do. All projects generated from the light-codegen will include a custom runtime, and the Gradle build script will call the graalvm docker to build the lambda functions to the native images.

* [native-image][] Native image with custom runtime to avoid cold start latency


[Lambda-Proxy]: /style/light-aws-lambda/lambda-proxy/
[Lambda-Framework]: /style/light-aws-lambda/lambda-framework/
[Lambda Proxy Generation]: /style/light-aws-lambda/codegen-proxy/
[Lambda Framework Generation]: /style/light-aws-lambda/codegen-framework/
[native-image]: /style/light-aws-lambda/native-image/

