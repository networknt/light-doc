---
title: "Body Sanitization"
date: 2023-11-06T18:21:53-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

### Introduction

In the [sanitizer][] document, we have provided general information about sanitizer.yml configuration and sanitization for the request header and body. All the functionalities are implemented in the [SanitizerHandler](https://github.com/networknt/light-4j/blob/master/sanitizer/src/main/java/com/networknt/sanitizer/SanitizerHandler.java) for services built with light-4j frameworks. 


However, when the SanitizerHandler is used in the light-gateway or http-sidecar, the body sanitization will be different due to request body interception. The [header sanitization][] is the same, though. 


### Body Sanitizer Plugin

When users build a service with one of the light-4j frameworks, the [BodyHandler][] will be used in the request/response chain. It will parse the request body based on the content type header and attach the JSON-based request body in the exchange. The subsequent SanitizerHandler will use the attached request body in the exchange. 


In the light gateway and HTTP sidecar, we need to transfer the request body from one socket to another socket. So, parsing the request body with the [BodyHandler][] is not an option as it will consume the request stream. To intercept the request body and keep the stream for the downstream service, we have implemented the [RequestBodyInterceptor](https://github.com/networknt/light-4j/blob/master/body/src/main/java/com/networknt/body/RequestBodyInterceptor.java).


This interceptor and [RequestTransformerInterceptor](https://github.com/networknt/light-4j/blob/master/request-transformer/src/main/java/com/networknt/reqtrans/RequestTransformerInterceptor.java) are called by the [RequestInterceptorInjectionHandler](https://github.com/networknt/light-4j/blob/master/handler/src/main/java/com/networknt/handler/RequestInterceptorInjectionHandler.java) in sequence. The RequestInterceptorInjectionHandler is defined in the request/response chain for the light-gateway and http-sidecar.

The RequestBodyInterceptor captures and attaches the body to the exchange, and the RequestTransformerInterceptor can call the plugin [body-sanitizer][] based on the [yaml-rule][] to encode the request body based on the sanitizer.yml configuration. 


### Configuration

For detailed configuration, please refer to the README.md in the [body-sanitizer][] plugin. There is also a video tutorial linked in the README.md to demo how the feature works. 


[sanitizer]: /concern/sanitizer/
[header sanitization]: /service/gateway/header-sanitization/
[BodyHandler]: https://github.com/networknt/light-4j/blob/master/body/src/main/java/com/networknt/body/BodyHandler.java
[yaml-rule]: https://github.com/networknt/yaml-rule
[body-sanitizer]: https://github.com/networknt/yaml-rule-plugin/tree/master/body-sanitizer

