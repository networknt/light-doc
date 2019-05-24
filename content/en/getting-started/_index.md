---
title: Getting Started
linktitle: Get Started Overview
description: Quick start and guides for staring your first microserivce on your preferred operating system.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
categories: [getting started,fundamentals]
keywords: [usage,docs]
menu:
  docs:
    parent: "getting-started"
    weight: 1
weight: 40
sections_weight: 40
draft: false
aliases: [/overview/introduction/]
toc: false
reviewed: true
---

Light-4j frameworks provide [middleware handlers][] to address cross-cutting concerns for your application so that you only need to implement the final [business handlers][] to process the request and return a response without thinking about exception handling, metrics, traceability, correlation, security, validation, etc. If you have the specification available, the [light-codegen][] can scaffold the application for you to fill in the business logic into the generated handlers and write assertions in the generated tests for these handlers. 

### Local Development Environment

If you don't have JDK 8 and Git installed, please prepare your [environment][]. 

### Choose a framework

There are several frameworks available to support different styles of API. If this is the very first time you try to create an API, please try [light-rest-4j][]. If you are unsure which style you should choose for your project, please visit [API category][].

* [Light-4j][] - For an expert that need absolute high throughput and low latency
* [Light-rest-4j][] - Restful API with OpenAPI 3.0 or Swagger 2.0 specification
* [Light-graphql-4j][] - GraphQL API with query, mutation and subscription based on IDL
* [Light-hybrid-4j][] - RPC style of hybrid monolithic and microservices serverless framework
* [Light-tram-4j][] - Guaranteed transactional message/commmand/event delivery between services
* [Light-eventuate-4j][] - An eventual consistency framework based on Event Sourcing and CQRS
* [Light-saga-4j][] - A saga implementation to manage distributed transaction across multiple microservices
* [Light-spring-boot][] - Leverage light-4j middleware handlers in existing Spring Boot application
* [Web Server][] - A web server that serve static content and APIs
* [WebSocket][] - A WebSocket server to support client to client or client to server communication
* [Light-session-4j][] - Distributed session managers (Redis, Hazelcast, JDBC) that support server cluster
* [Light-spa-4j][] - A set of middleware handlers for Single Page Application

### Set up an infrastructure service

Services we built

* [Light-oauth2][] - OAuth 2.0 provider implemented as microservices for microservices
* [Light-portal][] - A portal UI that integrate all the services to manage the life-cycle of services
* [Light-router][] - Working with external client or client not implemented in Java 8 and above
* [Light-proxy][] - Deploy in front of legacy API application to address cross-cutting concerns
* [Light-config-server][] - Centralized configuration and secret management service
* [Light-tokenization][] - Tokenize sensiteve info before sending info outside of corporate network

Services we integrated

* [Consul][] - For service registration and discovery
* [Kafka][] - Message broker that is most popular with event based frameworks
* [Elasticsearch][] - Centralized logging index and analysis
* [InfluxDB][] - Push the metrics data to the time series database
* [Prometheus][] - Pull the metrics data from each individual service
* [Kubernetes][] - Vanilla Kubernetes cluster platform
* [Openshift][] - Commercial Kubernetes cluster platform

### Leverage a tool

Tools we built

* [Light-bot][] - A microservice based DevOps agent that handles multiple repositories and dependencies
* [Light-codegen] - A code generator based on the rocker that can be used as a utility or web service
* [Swagger-bundler][] - A utility merges multiple Swagger spec files into a file with external references resolved
* [Openapi-bundler][] - A utility merges multiple OpenAPI spec files into a file with external references resolved

Tools we used

* [Wrk][] - Performance test tool that can generate enough requests per second

### Consume API or service

[Service Consumer][] - Client to service or service to service communication


### Getting Started

The easiest way to start your RESTful API project is from Swagger 2.0 specification or OpenAPI 
3.0 specification and here is a video to show you how to generate a project from Swagger spec.

[light-4j-getting-started](https://youtu.be/xSJhF1LcE0Q)


And here is the [step by step REST tutorial][] one walking through [Swagger petstore][] example and
the other is a [OpenAPI petstore][] example.

If you are interested in GraphQL framework, please refer to [light-graphql-4j][].

We also have a RPC/Hybrid framework that support JSON RPC and you can find some info at
[light-hybrid-4j][].

For event-driven service, you might be interested in [light-eventuate-4j][].


### Examples

Anther way to start a new project is to copy from an existing example and expends it. This 
is not recommended but if you don't have IDL(Inteface Definition Language) specification like 
OpenAPI specification or your service has very special requirement that cannot be generated, you 
can find many example projects at a separate repo [light-example-4j][] and start by copying one 
of them.

There are folders in light-example-4j that contains examples like:

* rest folder container examples built on top of light-rest-4j framework
* graphql folder contains examples built on top of light-graphql-4j framework
* hybrid folder contains examples built on top of light-hybrid-4j framework
* eventuate folder contains examples built on top of light-eventuate framework

**To get started, check out the [Quick Start][] page, to gain a general knowledge of how our Light-4j platform can be used and built upon.**


### Tutorials

Follow the [microservices tutorial][] to build multiple services that communicate each other. The 
source code for the tutorial can be found at [https://github.com/networknt/light-example-4j]

* api_a is calling api_b and api_c
* api_b is calling api_d
* api_c is an independent service
* api_d is an independent service


[step by step REST tutorial]: /tutorial/rest/
[Swagger petstore]: /tutorial/rest/swagger/petstore/
[OpenAPI petstore]: /tutorial/rest/openapi/petstore/
[light-example-4j]: https://github.com/networknt/light-example-4j
[Quick Start]: /tutorial/rest/swagger/ms-chain/quickstart
[microservices tutorial]: /tutorial/

[environment]: /getting-started/environment/
[API category]: /architecture/category/
[middleware handlers]: /architecture/middleware-handler/
[business handlers]: /architecture/business-handler/
[Light-4j]: /style/light-4j/
[Light-rest-4j]: /style/light-rest-4j/
[Light-graphql-4j]: /style/light-graphql-4j/
[Light-hybrid-4j]: /style/light-hybrid-4j/
[Light-tram-4j]: /style/light-tram-4j/
[Light-eventuate-4j]: /style/light-eventuate-4j/
[Light-saga-4j]: /style/light-saga-4j/
[Light-spring-boot]: /style/light-spring-boot/
[Web Server]: /style/webserver/
[WebSocket]: /style/websocket/
[Light-session-4j]: /style/light-session-4j/
[Light-spa-4j]: /style/light-spa-4j/

[Light-oauth2]: /service/oauth/
[Light-portal]: /service/portal/
[Light-router]: /service/router/
[Light-proxy]: /service/proxy/
[Light-config-server]: /service/config/
[Light-tokenization]: /service/tokenization/

[Consul]: /service/consul/
[Kafka]: /service/kafka/
[Elasticsearch]: /service/elasticsearch/
[Grafana]: /service/grafana/
[InfluxDB]: /service/influxdb/
[Prometheus]: /service/prometheus/
[Kubernetes]: /service/kubernetes/
[Openshift]: /service/openshift/


[light-codegen]: /tool/light-codegen/
[Light-bot]: /tool/light-bot/
[Swagger-bundler]: https://github.com/networknt/swagger-bundler
[Openapi-bundler]: https://github.com/networknt/openapi-bundler
[Wrk]: /tool/wrk-perf/
[Service Consumer]: /consumer/
