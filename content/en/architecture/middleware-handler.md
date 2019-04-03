---
title: "Middleware Handler"
date: 2019-03-08T11:27:10-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-4j and all related frameworks are trying to address all technical cross-cutting concerns for your applications so that developers won't need to worry about these concerns in their business logic. This significantly increases the productivity of the developers compare with other frameworks developers need to mix in all cross-cutting concerns in the business logic through annotations or method invocations. 

![light-4j-service-mesh-plus](/images/light-4j-service-mesh-plus.png)

In a request/response chain, request middleware handlers are injected between the client and [buisness handlers][] and response middleware handlers are injected between the [business handlers][] and the client. They are all running in the same server process, so the in-process communication between different handlers is super fast. 

Middleware handler means something plugged in the middle, and it collects, validates or manipulates request and/or response in the chain without returning anything to the client except error [status][]. In the request chain, once a middleware handler completes its process, it will pass the request to the next middleware handler until the business handler is reached to produce a response. Then the response will pass through a list of middleware handlers in the response chain until the response reaches the client. 

Most middleware handlers are either request handlers or response handlers, so they only work with request or response. However, there are some middleware handlers like metrics or audit which need to be working on both request and response to collect information. 

As you can see from the diagram above, there are two different types of middleware handlers:

* Technical Middleware Handler - They are generic handlers that work at the technical level and can be shared between applications across different industries or business domains.  Light-4j is aiming to provide all technical middleware handers. 

* Business Middleware Handler - They are running within the business context and are designed to be specific to a particular business domain. Most of our customers are building their own framework on top of light-4j by adding business middleware handlers for their organization. For example, a bank might implement its fine-grained authorization and business audit middleware handlers to address these concerns for all their services. 

For more information on how to use, customize or create middleware handlers, please visit [handler][] module.

[buisness handlers]: /architecture/business-handler/
[status]: /concern/status/
[handler]: /concern/handler/
