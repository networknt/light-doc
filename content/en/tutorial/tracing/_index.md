---
title: "Tracing"
date: 2019-06-15T19:57:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When moving monolithic architecture to monolithic, the majority of operational challenges that arise are narrowed in two areas: networking and observability. 

This is where Distributed Tracing can really help. Distributed Tracing helps with two fundamental challenges faced by microservices:

* Latency tracking

One user request or transaction can travel through many different services in different runtime environments. Understanding the latency of each of these services for a particular request is critical to the understanding of the overall performance characteristics of the system as a whole, and provides valuable insight for possible improvements.


* Root cause analysis

Root cause analysis is even more challenging for applications that build on top of large ecosystems of microservices. Anything can go wrong with any of the services at any time. Distributed tracing is of crucial importance when debugging issues in such a system.


Tracing is only one piece of the puzzles of the Three Pillars of Observability - Logging, Metrics and Tracing. 

In light-4j, we have built-in Traceability and Correlation handlers. Now, we have added OpenTracing support to give users more options. 

- [Jaeger](/tutorial/tracing/jaeger/)
- [Skywalking](/tutorial/tracing/skywalking/)