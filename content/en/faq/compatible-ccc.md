---
title: "Compatible Cross-Cutting Concerns"
date: 2018-03-14T21:37:39-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

How many cross-cutting concerns need to be implemented in order to bring services built with other frameworks or 
languages into light ecosystem?

As of today, Light only supports Java but some big customers have multiple LOBs with different preferred
languages or frameworks. We were constantly asked how to bring services built with other languages or frameworks
to the light ecosystem so that these services can interact with each other. 

We have a long term plan to provide Nodejs, Go, DotNet Core and Rust frameworks but it will take time. Meanwhile,
the following is a list of cross-cutting concerns that are mandatory. 

* Security Verification

Light utilizes OAuth 2.0 JWT tokens with scopes for security. A security validator needs to be implemented
to verify the signature of the JWT token as well as comparing specification defined scopes with JWT token scopes.

This means that the service needs to load the specification during runtime in order to do the comparison. It also
needs to have a plugin architecture to support fine-grained authorization within business context based ont the
custom claims in the JWT token. 

For maximum security, two tokens (subject token and access token) need to be supported at the very least. Token exchange and token chaining are optional.

* Specification Validation    

Before the request reaches the business handler, it needs to be validated based on the specification. Only valid requests will reach the business handler. The error handling is fail fast and this reduces the attack surface, meaning it won't expose the business handler to an invalid request.  

* Traceability and Correlation

The service needs to generate correlationId and pass traceabilityId (If passed in) and correlationId to the next service. All logging statements should have the traceabilityId and correlationId in order to diagnose production
issues. 

* Centralized Logging

Logging need to be sent to a centralized server for indexing, alertd, monitoring and searches. TraceabilityId and correlationId
will help to link logs for the same external request together. 

* Centralized Metrics

Metrics need to be collected and send to a centralized time serial database. The information needs to be collected
from both consumer perspective and provider perspective. Multiple dimensional data need to be collected in each category.

* Request Auditing

For enterprise level services, a centralized auditing is needed. It can be a separate log file or database and it is
better to give an plugin interface so that different team can choose their implementation. 

* Exception Handling

There must be an exception handler to capture all exceptions and convert them to formatted error messages. This includes
both checked exception and runtime exception. It is not allowed to return a 500 error with stacktrace to the caller.

* Service Registration

Service needs to register itself to a centralized registry so that consumers can find it. There are vary tools for service
registration but Consul is the most popular one. 


* Consumer support

We need to provide a consumer module which can do service discovery, client side load balance and cluster support. The module
is responsible for retrieving/renewing JWT token from OAuth 2.0 provider. It is best for the consumer module to support
HTTP 2.0 in order to leverage the high performance with other services. 

* Health Check

Every service needs to expose a health check endpoint that the registry service can pull to maintain a list of healthy
instances. If Kubernetes Ingress is used, it needs this endpoint as well for load balancing.

* Encryptor Decryptor

This is a pair of an encryptor and decryptor to hide password or passphrase in config files. 

The above are mandatory, but here is a list with optional cross-cutting concerns. They are not mandatory for all services, but it is nice to have for most of the services. 

* Mask

An utility that can mask sensitive info like credit card etc. before logging into file system based logs or database logs. 

* Sanitizer

Remove cross site scripting risks from incoming request for certain APIs.

* Rate limiting

For Internet facing APIs

* CORS

Cross-Origin Resource Sharing support to handle the in-flight options request.

* Externalized config

To change the configuration of service outside of docker container. 

There are a lot of details on each [cross-cutting concerns][]. If you are working on a framework that want to integrate
with light-platform, we can help or work together with you.  


[cross-cutting concerns]: /concerns/

