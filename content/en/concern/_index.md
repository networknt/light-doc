---
title: "Concerns Overview"
date: 2017-11-04T22:04:22-04:00
description: "Plugins or Middleware Handlers that address all the cross-cutting concerns"
categories: [concerns,fundamentals]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 01
weight: 50	#rem
sections_weight: 50
aliases: []
toc: false
draft: false
---

One of the design goals of the light-4j is to address all the technical cross-cutting concerns in the frameworks so that service developers will only focus on the business logic without worry about security, auditing, logging, metrics, etc. The same pattern can be applied in the business context as well so that you can have several handlers to perform different tasks within the business context. For example, the fine-grained authorization can be done as a cross-cutting concern in the business context without mixing into the real business logic within the core request handler. 

## Technical Cross-Cutting Concerns

When building an application, there is a functional requirement, and normally there is always a non-functional requirement that is created to address all technical concerns. In most platforms, developers need to address these concerns in the same business handler at the same time. It makes the application very hard to maintain and to reason about as functional requirements, and business requirements are implemented at the same time in the same context. The code is blended and hard to separate, hence decrease the maintainability. The light platform divides these concerns into individual handlers so that multiple developers can work together. These handlers are relatively simple and easy to be shared between services, and we have provided dozens built-in handlers in the platform. Without non-functional concerns in the business logic, the main business handler will be much simpler to implement and reason about. In a long run, it is easy to maintain as any modification will be focused on the business logic only. 

Here is a list of cross-cutting concerns provided by the light platform. 

#### light-4j

* [Audit log](/concern/audit/)
* [Load Balance](/concern/balance/)
* [Basic Authentication](/concern/basic/)
* [Body Parser](/concern/body/)
* [HTTP/2 Client](/concern/client/)
* [Cluster Support](/concern/cluster/)
* [Common Component](/concern/common/)
* [Extenal Config](/concern/config/)
* [Consul Client](/concern/consul/)
* [Correlation Id](/concern/correlation/)
* [CORS Handler](/concern/cors/)
* [Data Source](/concern/datasource/)
* [Decryptor](/concern/decryptor/)
* [Request Dump](/concern/dump/)
* [Encode Decode](/concern/encode-decode/)
* [Exception Handling](/concern/exception/)
* [Handler Interface](/concern/handler/)
* [Business Handler](/concern/business-handler/)
* [Header Handler](/concern/header/)
* [Health Check](/concern/health/)
* [Server Info](/concern/info/)
* [IP Whitelist](/concern/ip-whitelist/)
* [Rate Limit](/concern/limit/)
* [Log Mask](/concern/mask/)
* [Logger Config](/concern/logger-config/)
* [InfluxDB Metrics](/concern/metrics/)
* [Portal Registry](/concern/portal-registry/)
* [Prometheus Metrics](/concern/prometheus/)
* [Registry Discovery](/concern/registry/)
* [XSS Sanitizer](/concern/sanitizer/)
* [Security](/concern/security/)
* [Server](/concern/server/)
* [Service](/concern/service/)
* [Status](/concern/status/)
* [Switch](/concern/switch/)
* [Traceability Id](/concern/traceability/)
* [Utility](/concern/utility/)
* [ZooKeeper Client](/concern/zookeeper/)

#### light-rest-4j

* [OpenAPI Meta](/style/light-rest-4j/openapi-meta/)
* [OpenAPI Security](/style/light-rest-4j/openapi-security/)
* [OpenAPI Validator](/style/light-rest-4j/openapi-validator/)
* [Swagger Meta](/style/light-rest-4j/swagger-meta/)
* [Swagger Security](/style/light-rest-4j/swagger-security/)
* [Swagger Validator](/style/light-rest-4j/swagger-validator/)

#### light-graphql-4j

* [GraphQL Common](/style/light-graphql-4j/graphql-common/)
* [GraphQL Router](/style/light-graphql-4j/graphql-router/)
* [GraphQL Security](/style/light-graphql-4j/graphql-security/)
* [GraphQL Validator](/style/light-graphql-4j/graphql-validator/)

#### light-hybrid-4j

* [Binary RPC](/style/light-hybrid-4j/binary-rpc/)
* [JSON RPC](/style/light-hybrid-4j/json-rpc/)
* [RPC Router](/style/light-hybrid-4j/rpc-router/)
* [RPC Security](/style/light-hybrid-4j/rpc-security/)

#### light-aws-lambda

With Lambda framework


With Light-Proxy



## Business Cross-Cutting Concerns

In a system, besides of technical concerns, there are other concerns that need to be addressed within the business context. In most systems, these will be blended into the business logic implementation which makes the business rules hard to reason about. In our design, these are separated into individual handlers which wired in before or after the main business handlers. For example, fine-grained authorization based on the custom claim in the JWT token or result filter based on the client_id in the JWT token. 

