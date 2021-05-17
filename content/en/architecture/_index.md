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
reviewed: true
---

The light-4j is aimed towards microservices, and therefore has to achieve high throughput, low latency, light-weight and address a lot of cross-cutting concerns at the same time. It is based on Undertow Core Http server and depends on minimum third-party libraries.

The topics here are architectural considerations. In a nutshell, architecture is a type of design where the focus is quality attributes and wide(er) scope whereas design focuses on functional requirements and more localized concerns. There is another [design][] section for detail-oriented decisions.

Here is a list of architecture decisions for the platform:

* Designed for [microservices][] that can be dockerized and deployed within containers.

* Based on pure HTTP without JavaEE as it has too many problems and is [declining][].
 
* [Security][] first design with OAuth2 integration and distributed verification with the embedded distributed gateway.

* The platform handles the [coarse-grained authorization with scopes and fine-grained authorization with custom claims][]. 

* How to handle distributed [transactions][] in microservices.

* All components are designed as [plugins][] and the framework is easy to be extended and customized.

* Can be [integrated][] with existing applications to protect investment over the years for your organization.

* Support [client-side discovery][] without any [gateway][] and proxy between service to service calls as they add too much overhead. 

* Service logs will be aggregated with ElasticSearch, LogStash and Kibana with [monitoring and alerting][].

* Built-in [CorrelationId and TraceabilityId][] to trace service to service calls in aggregated logs.

* Designed for [scalability][] so that you can have thousands of instances running at the same time. 
 
* Has its own dependency injection framework so that developers can avoid heavy Spring or Guice as they are [bloated][]. 

* [Elements][] of API platform form an eco-system. 

* API [category][] and how to choose a framework to build your APIs.

* What is a [service mesh][] and why do we need it? 

* We cannot rely on network security for microservices given the dynamic environment and encrypted data. [Firewall is killed by Cloud][] 

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
[service mesh]: /architecture/service-mesh/
[client-side discovery]: /architecture/client-discovery/
[Firewall is killed by Cloud]: /architecture/firewall/
[coarse-grained authorization with scopes and fine-grained authorization with custom claims]: /architecture/cga-fga/
