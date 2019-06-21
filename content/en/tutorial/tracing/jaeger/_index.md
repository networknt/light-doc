---
title: "Jaeger OpenTracing"
date: 2019-06-15T19:58:06-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The OpenTracing API provides a standard, vendor neutral framework for instrumentation. This means that if a developer wants to try out a different distributed tracing system, then instead of repeating the whole instrumentation process for the new distributed tracing system, the developer can simply change the configuration of the Tracer.

Please visit [OpenTracing](https://opentracing.io/) site for more information.

There are numeric tracers implement OpenTracing API and one of them is CNCF Jaeger. In this tutorial, we are going to walk you the steps to leverage Jaeger with a distributed light-4j application. 

In light-4j, we have a new module called jaeger-tracing added with a JaegerStartupHookProvider to init the tracer and a JaegerHandler to extract the Tracing context and start the Span and finish the Span. 

The client module is also modified to add two more public method to inject the tracing context to the next service in the request. 

* [Start Jaeger Tracer](/tutorial/tracing/jaeger/all-in-one/)
* [Petstore Jaeger](/tutorial/tracing/jaeger/petstore-jaeger/)
* [Service Discovery](/tutorial/tracing/jaeger/service-discovery/)

