---
title: Why Light Platform
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
layout: single
menu:
  docs:
    parent: "about"
    weight: 10
weight: 10
sections_weight: 10
draft: false
toc: true
reviewed: true
---

* HTTP sidecar can be utilized as a sidecar along with the backend API in the same pod in a Kubernetes cluster. It allows backend API can be implemented with any language or framework without cross-cutting concerns. 

* Light Gateway depends on the same codebase as the HTTP sidecar but focuses on the standalone deployment for consumer router, provider proxy and shared gateway. 

* Kafka sidecar works with backend application or API to produce and consume messages to and from Kafka and host streams application.

* APIs built with the light-4j frameworks can select gateway middleware handlers in the same instance without HTTP sidecar to reduce the size and simplify the deployment.

* All the above can be managed with the light controller to shut down, restart, reload configuration, register or discover service, change logging level and trigger chaos monkey attacks. 

* All light-4j components and middleware handlers have their individual configuration with default templates in each module. Users can use values.yml to overwrite default values, and all config files can be loaded from light-config-server during the server startup.

* Build both synchronous request/response service and asynchronous event-based service with the same platform with uniformed security, validation, tracing, audit and metrics.  

* Compile the application with GraalVM to the native image to reduce memory usage multiple times. 

* Built-in RuleEngine-based request/response transform interceptors to transform the request and response easily and openly. 

* Built-in fine-grained role-based, attribute-based and rule-based authorization and entitlement management.

* An open architecture that allows users to customize existing middleware handlers and add new middleware handlers to address specific business concerns.

* Light platform covers all aspects of enterprise architecture and provides a total solution with integration with other third-party systems via customized handlers.


