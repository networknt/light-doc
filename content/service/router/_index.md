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
---

All the services developed on top of light*4j frameworks support [client side service discovery](http://microservices.io/patterns/client-side-discovery.html), 
load balance and cluster natively. However, for some of the client that cannot leverage our client
module, we have provide light-router which basically move the client module on the network to allow
the original client to consume service as it is today without considering how dynamic these services
in the cloud. 

Along with service discovery and routing, light-router also provide other cross-cutting concerns
like security etc.

The light-router has the following features:

* High throughput, low latency and small footprint. 
* Service discover with Consul, Zookeeper etc.
* Support environment tags for multiple testing environments deployed to the same cloud.  
* Integrate light-oauth2 to retrieve the access token and renew the token before expiration.
* Built-in load balancer and cluster support
* Can be started with Docker or standalone
* Support HTTP 2.0 protocol on both in/out connections
* TLS termination
* Support REST, GraphQL and RPC style of APIs
* Centralized logging with ELK, traceabilityId and CorrelationId
* Collect client and service metrics into InfluxDB and view the dashboard on Grafana
* Manage configuration with light-config-server

To learn how to use this proxy, pleases refer to 

* [Getting Started][] to learn why we have provide this router
* [Tutorial][] with step by step guide for RESTful router
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations


  
[Getting Started]: /getting-started/light-router/
[Tutorial]: /tutorial/router/
[Configuration]: /service/router/configuration/
[Artifact]: /service/router/artifact/
