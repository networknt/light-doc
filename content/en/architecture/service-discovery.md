---
title: "Service Discovery"
date: 2018-02-14T09:12:55-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 100
weight: 100
aliases: []
toc: false
draft: false
---

When introducing microservices architecture to an organization, one of the important decisions
that needs to be made is client side service discovery or service side discovery. 



## Who is using client side discovery

* In Java world, most people are using Spring Boot to build their services with Netflix open source
libraries(Eureka for service registry and Ribbon is a jar include inside consumer)

* Light-4j client side discovery is derived from an RPC open source platform motan. 
https://github.com/weibocom/motan

* https://grpc.io/blog/loadbalancing

* http://techorgan.com/microservices/microservices-series-a-quick-look-at-service-discovery-patterns/


## Service Mesh

* http://philcalcado.com/2017/08/03/pattern_service_mesh.html

* https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/

* https://www.hashicorp.com/blog/smart-networking-with-consul-and-service-meshes


