---
title: "What is light-4j"
date: 2017-12-21T06:34:12-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

light-4j is a platform or ecosystem for building and running cloud native microservices. The
design goal is higher throughput, lower latency and smaller memory footprint to lower production cost.

It contains the following: 

### Cross-cutting concerns

All light-4j frameworks are built on top of an embedded gateway to address cross-cutting concerns
for cloud native services within the request/response chain. These plugins or middleware handlers 
are wired in with IoC service during server startup and can be enabled or disabled or change 
behaviors via configurations. One of the design goal is to allow developers only write business logic.

For more info please refer to [cross-cutting concerns][]


### Interaction Style

In microservices architecture, services need to interact with each other and there are several
frameworks to help build services with certain interaction style. 

Synchronous (Request/Response over HTTP): 

[light-rest-4j][] - Building restful API and supporting Swagger 2.0 and OpenAPI 3.0 specification

[light-graphql-4j][] - Generating GraphQL service with IDL

[light-hybrid-4j][] - RPC and Serverless framework that takes advantages of both monolithic and micorservices.

Asynchronous (Event Driven): 

[light-tram-4j][] - Transactional messaging to ensure delivery of message, event or command.

[light-eventuate-4j][] - Event sourcing and CQRS framework
 
[light-saga-4j][] - Distributed transaction orchestration across microservices.

### Infrastructure service

To support microservices, the initial infrastructure services must be implemented first. It
includes:

[light-oauth2][] - An OAuth 2.0 provider implemented as microservices. 

[light-portal][] - An API management portal and marketplace (Work in progress)

ELK - Centralized logging

InfluxDB and Grafana - Centralized metrics

Consul - Service registry and discovery

Kafka and Zookeeper - Message Broker


### Tool Chain

[light-codegen][] - Code generator for all frameworks. 

[openapi-parser][] - A light weight and fast OpenAPI 3.0 parser and validator

[swagger-bundler][] - Bundle multiple swagger files together and validate the final result.

[light-bot][] - Devops pipeline for microservices.


[cross-cutting concerns]: /concern/
[light-rest-4j]: /style/light-rest-4j/
[light-graphql-4j]: /style/light-grahpql-4j/
[light-hybrid-4j]: /style/light-hybrid-4j/
[light-tram-4j]: /style/light-tram-4j/
[light-eventuate-4j]: /style/light-eventuate-4j/
[light-saga-4j]: /style/light-saga-4j/
[light-oauth2]: /service/oauth/
[light-portal]: https://github.com/networknt/light-portal
[light-codegen]: /tool/light-codegen/
[openapi-parser]: /tool/openapi-parser/
[swagger-bundler]: /tool/swagger-bundler/
[light-bot]: https://github.com/networknt/light-bot
