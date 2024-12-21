---
title: "Request Body Interceptor"
date: 2023-01-13T12:44:48-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Request Body Interceptor is an interceptor implementation that is injected into the request/response chain via the [RequestInterceptorInjectionHandler][]. There are to light-4j built-in request interceptors. 

[RequestBodyInterceptor][] is used to intercept the request body in the http-sidecar and light-gateway to support the body transformation and log the request body into the audit log for debugging. 

[RequestTransformerInterceptor][] is used to transform the request body to another format in order to bridge the difference between the consumer and provider. 



[RequestInterceptorInjectionHandler]: /concern/request-interceptor-injection/
[RequestBodyInterceptor]: /concern/request-body-interceptor/
[RequestTransformerInterceptor]: /concern/request-transformer-interceptor/
