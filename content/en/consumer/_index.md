---
title: "Service Consumer"
date: 2018-03-01T16:44:15-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "consumer"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Majority of the sections in this site are about how to build, deploy and manage
services. This section will address microservices architecture and best practices 
from consumer perspective. 

Unlike monolithic JavaEE web service with only one client and one server, there
are many microservices involved in one external request and client to server
communication efficiency is very important to overall latency. The definition
of client vs server or consumer vs provider is blurred as one service can be 
the server for another service and at the same time it can be a client for
another service. 

The following contains architecture, design and best practices. 

- [Client Module](/consumer/client-module/)
  * [Which Jar Files](/consumer/jar-files/)
  * [TLS Connection](/consumer/tls-connection/)
  * [HTTP 2.0](/consumer/http2/)
  * [Security](/consumer/security/)
     + [Subject Token](/consumer/subject-token/)
     + [Access Token](/consumer/access-token/)
     + [Customized Grant Type](/consumer/customized-grant/)
     + [Get Token in Startup](/consumer/token-startup/)
     + [Long Lived Token](/consumer/long-lived-token/)
     + [Key Distribution](/architecture/key-distribution/)
     + [Secret Encryption](/consumer/secret-encryption/)
     + [SPA Stateful](/consumer/spa-session-jwt/)
     + [SPA Stateless](/consumer/spa-cookie-jwt/)
     + [SPA CSRF Prevention](/consumer/spa-csrf/)
     + [SPA XSS Prevention](/consumer/spa-xss/)
     + [SPA BFF](/consumer/spa-bff/)
  * [Traceability](/consumer/traceability/)
     + [Traceability Id](/consumer/traceability-id/)
     + [Correlation Id](/consumer/correlation-id/)
  * [Circuit Breaker](/consumer/circuit-breaker/)
  * [CompletableFuture](/consumer/completable-future/)
- [light-consumer-4j](/consumer/light-consumer-4j/)
- [Service Discovery](/consumer/service-discovery/)
  * [Registry](/consumer/registry/)
     + [Standalone](/consumer/standalone-registry/)
     + [Docker](/consumer/docker-registry/)
     + [Kubernetes](/consumer/kubernetes-registry/)
  * [Discovery](/consumer/discovery/)
     + [Consul](/consumer/consul-discovery/)
     + [Zookeeper](/consumer/zookeeper-discovery/)
  * [SPA and Mobile](/consumer/spa-mobile/)   
- [Load Balance](/consumer/load-balance/)
  * [Round Robin](/consumer/round-robin/)
  * [Local First](/consumer/local-first/)
  * [Consistent Hash](/consumer/consistent-hash/)
- [Light Router](/consumer/light-router/)
  * [Use Cases](/consumer/router-use-case/)
  * [Location and Ownership](/service/router/location-ownership/)
- [Single Page Application](/consumer/spa/)
  * [React](/consumer/react/)
  * [React-schema-form](/consumer/react-schema-form/)
  * [Vue](/consumer/vue/)
  * [Angular](/consumer/angular/)
