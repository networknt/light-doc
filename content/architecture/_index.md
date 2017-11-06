---
title: "Architecture Overview"
date: 2017-11-06T11:14:26-05:00
description: "Light Platform Architecture"
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

light-4j is aiming microservices and it has to be high throughput, low latency, light-weight and
address a lot of cross-cutting concerns at the same time. It is based on Undertow Core Http server
and depending on minimum third party libraries.

The topics here are architectural considerations. In a nutshell, architecture is a type of design
where the focus is quality attributes and wide(er) scope whereas design focuses on functional
requirements and more localized concerns. There is another [design][] section for detail oriented 
decisions.

Here is a list of architecture decisions for the framework:

* Designed for [microservices][] that can be dockerized and deployed within containers.

* Base on pure HTTP without JavaEE as it has too many problems and is [declining][].
 
* [Security][] first design with OAuth2 integration and distributed verification with embedded distributed gateway.

* How to handler distributed [transaction][] in microservices.

* All components are designed as [plugins][] and the framework is easy to be extended and customized.

* Can be [integrated][] with existing application to protect investment over the year for your organization.

* Support direct calls from microservice to microservice without any [gateway][], proxy as they add too much overhead. 

* Service logs will be aggregated with ElasticSearch, LogStash and Kibana with [monitoring and alerting][].

* Built-in [CorrelationId and TraceabilityId][] to trace service to service calls in aggregated logs.

* Designed for [scalability][] so that you can have thousands instances running at the same time. 
 
* Has its own dependency injection framework so that developers can avoid heavy Spring or Guice as they are [bloated][]. 

* [Elements][] of API platform. 

* API [category][] and how to choose a framework to build your APIs.


[design]: /design/
[microservices]: /architecture/microservices/
[declining]: /architecture/jee-is-dead/
[Security]: /architecture/security/
[transaction]: /architecture/transaction/
[plugins]: /architecture/plugin/
[integrated]: /architecture/integration/
[gateway]: /architecture/gateway/
[monitoring and alerting]: /architecture/monitoring/
[CorrelationId and TraceabilityId]: /architecture/traceability/
[scalability]: /architecture/scalability/
[bloated]: /architecture/spring-is-bloated/
[Elements]: /architecture/platform/
[category]: /architecture/category/
