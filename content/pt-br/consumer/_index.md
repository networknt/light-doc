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

- [Módulo cliente](/ptbr/consumer/client-module/)
  * [Arquivos jar](/ptbr/consumer/jar-files/)
  * [Conexão TLS](/ptbr/consumer/tls-connection/)
  * [HTTP 2.0](/ptbr/consumer/http2/)
  * [Segurança](/architecture/security/)
     + [Token de entidade](/ptbr/consumer/subject-token/)
     + [Token de acesso](/ptbr/consumer/access-token/)
     + [Tipo de concessão personalizado](/ptbr/consumer/customized-grant/)
     + [Obter token no Startup](/ptbr/consumer/token-startup/)
     + [Token de longa duração](/ptbr/consumer/long-lived-token/)
     + [Distribuição de chave](/architecture/key-distribution/)
     + [Criptografia secreta](/ptbr/consumer/secret-encryption/)
     + [SPA Stateful](/ptbr/consumer/spa-session-jwt/)
     + [SPA Stateless](/ptbr/consumer/spa-cookie-jwt/)
     + [SPA prevenção de CSRF](/ptbr/consumer/spa-csrf/)
     + [SPA prevença de XSS](/ptbr/consumer/spa-xss/)
     + [SPA BFF](/ptbr/consumer/spa-bff/)
  * [Rastreabilidade](/ptbr/consumer/traceability/)
     + [Id da Rastreabilidade](/ptbr/consumer/traceability-id/)
     + [Id de correlação](/ptbr/consumer/correlation-id/)
  * [Circuit Breaker](/ptbr/consumer/circuit-breaker/)
  * [CompletableFuture](/ptbr/consumer/completable-future/)
- [light-consumer-4j](/ptbr/consumer/light-consumer-4j/)
- [Service Discovery](/ptbr/consumer/service-discovery/)
  * [Registry](/ptbr/consumer/registry/)
     + [Standalone](/ptbr/consumer/standalone-registry/)
     + [Docker](/ptbr/consumer/docker-registry/)
     + [Kubernetes](/ptbr/consumer/kubernetes-registry/)
  * [Discovery](/ptbr/consumer/discovery/)
     + [Consul](/ptbr/consumer/consul-discovery/)
     + [Zookeeper](/ptbr/consumer/zookeeper-discovery/)
  * [SPA e móvel](/ptbr/consumer/spa-mobile/)   
- [Load Balance](/ptbr/consumer/load-balance/)
  * [Round Robin](/ptbr/consumer/round-robin/)
  * [Local First](/ptbr/consumer/local-first/)
  * [Consistent Hash](/ptbr/consumer/consistent-hash/)
- [Light Router](/ptbr/consumer/light-router/)
  * [Casos de uso](/ptbr/consumer/router-use-case/)
  * [Localização e propriedade](/service/router/location-ownership/)
- [Single Page Application](/ptbr/consumer/spa/)
  * [React](/ptbr/consumer/react/)
  * [React-schema-form](/ptbr/consumer/react-schema-form/)
  * [Vue](/ptbr/consumer/vue/)
  * [Angular](/ptbr/consumer/angular/)
