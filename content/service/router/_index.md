---
title: "Light Router"
date: 2018-02-11T16:29:11-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 8
weight: 8	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

All the services developed on top of light*4j frameworks support [client side service discovery](http://microservices.io/patterns/client-side-discovery.html), load balance and cluster natively. However, for some of the client that cannot leverage our client module, we have provided light-router which move the client module on the network to allow the original client to consume service as it is today without considering how dynamic these services in the cloud. 


Another usage for the light-router is to act as BFF(backend for frontend) for Single Page Applications or Mobile Native Applications. It supposed to be used only for simple use cases that client wants to access backend APIs built on top of the light platform. If there are additional logic in the business context, for example, transformation, orchestration, aggregation and integration, we need to create a microservice for BFF. 


Along with service discovery and routing, light-router also provide other cross-cutting concerns like security, metrics, logging, validation and more.

The light-router has the following features:

* High throughput, low latency and small footprint. 
* TLS passthrough or termination
* Service discovers with Consul, Zookeeper etc.
* Support environment tags for multiple testing environments deployed to the same cloud.  
* Integrate light-oauth2 to retrieve the access token and renew the token before expiration.
* Built-in load balancer and cluster support
* Can be started with Docker or standalone
* Support HTTP 2.0 protocol on both in/out connections
* Support REST, GraphQL and RPC style of APIs
* Centralized logging with ELK, TraceabilityId, and CorrelationId
* Collect client and service metrics into InfluxDB and view the dashboard on Grafana
* Manage configuration with light-config-server

To learn how to use this router, please refer to 

* [Getting Started][] to learn why we have provided this router
* [Tutorial][] with step by step guide for the RESTful router
* [Components][] router specific components and middleware handlers
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* [Location and Ownership] for deployment and ownership decision
  
[Getting Started]: /getting-started/light-router/
[Tutorial]: /tutorial/router/
[Components]: /service/router/components/
[Configuration]: /service/router/configuration/
[Artifact]: /service/router/artifact/
[Location and Ownership]: /service/router/location-ownership/
