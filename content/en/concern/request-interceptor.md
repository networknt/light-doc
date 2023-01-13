---
title: "Request Interceptor"
date: 2022-07-20T17:34:59-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

As we all know, middleware handlers are used to address cross-cutting concerns in the request/response chain. Some middleware handlers are used to manipulate the request or response, like the header handler to update the request and response headers based on the configuration file.

Although we have provided a lot of middleware handlers to help users to manipulate the request and response as cross-cutting concerns, we cannot predict all the use cases our users come up with. To help users to transform the request and response based on their specific use cases, we have implemented interceptor injection handlers for request and response manipulations. 

In this document, we are focusing on the request interceptor only. If you are interested in response interceptor, please click [here](/concern/response-interceptor/).

Users can implement multiple request interceptors, which will chain together as a sub-chain in middleware handlers' request/response chain. 

To inject a list of interceptors to the request/response chain, we need to add the  RequestInterceptorInjectionHandler to the default chain in the handler.yml or values.yml if the default chain is externalized like in the light-gateway and http-sidecar.

```
handler.handlers:
  # Light-framework cross-cutting concerns implemented in the microservice
  .
  .
  .
  - com.networknt.traceability.TraceabilityHandler@traceability
  - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.handler.RequestInterceptorInjectionHandler@requestInterceptor
  - com.networknt.header.HeaderHandler@header
  .
  .
  .

handler.chains.default:
  .
  .
  .
  - traceability
  - correlation
  - requestInterceptor
  - header
  .
  .
  .

```

Please note that the requestInterceptor is placed after the correlation and before any meaningful middleware handlers. 

Here is the built-in configuration file request-injection.yml for RequestInterceptorInjectionHandler in the module. It is enabled by default as long as it is in the handler chain. By default, the request body is not injected as it is very heavy. If you set up some request body transformers or want to log the request body into the audit log, you should add the request path prefix to appliedBodyInjectionPathPrefixes.

```
# request interceptor injection handler configuration

# indicator of enabled
enabled: ${request-injection.enabled:true}
# request body injection applied path prefixes. Injecting the request body and output into the audit log is very heavy operation,
# and it should only be enabled when necessary or for diagnose session to resolve issues. This list can be updated on the config
# server or local values.yml, then an API call to the config-reload endpoint to apply the changes from light-portal control pane.
# Please be aware that big request body will only log the beginning part of it in the audit log and gzip encoded request body can
# not be injected. Even the body injection is not applied, you can still transform the request for headers, query parameters, path
# parameters etc. The format is a list of strings separated with commas or a JSON list in values.yml definition from config server,
# or you can use yaml format in this file or values.yaml on local filesystem. The following are the examples.
# request-injection.appliedBodyInjectionPathPrefixes: ["/v1/cats", "/v1/dogs"]
# request-injection.appliedBodyInjectionPathPrefixes: /v1/cats, /v1/dogs
# request-injection.appliedBodyInjectionPathPrefixes:
#   - /v1/cats
#   - /v1/dogs
appliedBodyInjectionPathPrefixes: ${request-injection.appliedBodyInjectionPathPrefixes:}

```

To implement an interceptor, you must implement the following interface. 

```
package com.networknt.handler;

/**
 * This is the interface for the request interceptors. It is just a normal middleware handler with some extra
 * indicators.
 */
public interface RequestInterceptor extends MiddlewareHandler {
    boolean isRequiredContent();
}

```

This interface is in the handler module. If the isRequiredContent() is true, then the RequestInterceptorInjectionHandler will parse the request body and put it into the exchange as an attachment for the interceptor to use and modify. Otherwise, the interceptors will only work on other request attributes except for the request body. 

All interceptors are defined in the service.yml, and it can be externalized to the values.yml

```
# service.yml
service.singletons:
  .
  .
  .
  - com.networknt.handler.RequestInterceptor:
      - com.networknt.reqtrans.RequestTransformerInterceptor

```

The above RequestTransformInterceptor is an implementation of the RequestInterceptor based on the rule engine so that it can be very flexible to do anything with rule definition. 


Here is the config file for the RequestTransformerInterceptor

```
# request-transformer.yml
# indicate if this interceptor is enabled or not.
enabled: ${request-transformer.enabled:true}
# indicate if the transform interceptor needs to change the request body
requiredContent: ${request-transformer.requiredContent:true}
# A list of applied request path prefixes, other requests will skip this handler. The value can be a string
# if there is only one request path prefix needs this handler. or a list of strings if there are multiple.
appliedPathPrefixes: ${request-transformer.appliedPathPrefixes:}

```

When using it, you need to specify if the request body is needed for the transformer rule implementation. Also, you need to specify a list of prefixes in the request paths to limit the usage of the interceptor for all endpoints. 

Here is a [tutorial](/tutorial/gateway/request-response-transformation/) to show you how to use the request interceptors. 

