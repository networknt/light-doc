---
title: "Consumidor de serviço"
date: 2019-04-27T13:37:28-03:00
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
draft: true
---

Maioria dessas seções neste site são como construir, implantar e gerenciar serviços.
Esta seção discutirá arquitetura de microserviços e melhores práticas da perspectiva do consumidor.

Ao contrário de monolitos web services JavaEE com somente um cliente e um servidor, há muitos microserviços
envolvidos em uma requisição externa e é muito importante a latência geral na eficiência na comunicação cliente e servidor.

A definição de cliente vs servidor ou consumidor vs provedor é ofuscada como um serviço que pode ser o servidor para um outro serviço 
e ao mesmo tempo isto pode ser um cliente para outro serviço.

O menu abaixo contém arquitetura, design e melhores práticas: 

- [Client Module](/consumer/client-module/)
  * [Which Jar Files](/consumer/jar-files/)
  * [TLS Connection](/consumer/tls-connection/)
  * [HTTP 2.0](/consumer/http2/)
  * [Security](/architecture/security/)
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
