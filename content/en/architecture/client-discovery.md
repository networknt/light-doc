---
title: "Client-Side Discovery"
date: 2021-05-11T20:42:53-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Here is an article that discusses the [discovery patterns](https://www.javacodegeeks.com/2016/11/microservices-series-quick-look-service-discovery-patterns.html). You can learn what client-side vs server-side discovery is in the above article.


### Who is using client-side discovery

* In the Java world, most people are using Spring Boot to build their services with Netflix open source
libraries(Eureka for service registry and Ribbon is a jar include inside consumer)

* Light-4j client-side discovery is derived from an RPC open-source platform [motan](https://github.com/weibocom/motan). 

* Here is an article focusing on the [load balance](https://grpc.io/blog/loadbalancing) of gRPC in client-side discovery. 


When client-side discovery is used, there are two different deployment patterns: Standalone and Service Mesh. The [features](/architecture/service-discovery/) provided by the platform are the same in both patterns. 

## Standalone



## Service Mesh



* http://philcalcado.com/2017/08/03/pattern_service_mesh.html

* https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/

* https://www.hashicorp.com/blog/smart-networking-with-consul-and-service-meshes


