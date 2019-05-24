---
title: "Light Proxy"
date: 2017-12-16T09:21:13-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 7
weight: 7	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

All the services developed on top of light*4j frameworks support [client side service discovery](http://microservices.io/patterns/client-side-discovery.html), load balance and cluster natively. So there is no need to put a reverse proxy instance in front of our services like other API frameworks that support only [server side service discovery](http://microservices.io/patterns/server-side-discovery.html).

Also, light services embed a distributed gateway to address all the cross-cutting concerns in the request/response chain and work with the ecosystem that consists:

* [light-oauth2](https://github.com/networknt/light-oauth2) for security
* [light-portal](https://github.com/networknt/light-portal) for API management and market place
* [light-config-server](https://github.com/networknt/light-config-server) for centralized configuration management
* [light-eventuate-4j](https://github.com/networknt/light-eventuate-4j) for eventual consistency based on event sourcing, CQRS and Kafka
* [ELK](https://www.elastic.co/webinars/introduction-elk-stack) for centralized logging with traceabilityId and correlationId
* [InfluxDB](https://github.com/influxdata/influxdb) and [Grafana](https://github.com/grafana/grafana) for metrics
* [Consul](https://github.com/hashicorp/consul) or [Zookeeper](http://zookeeper.apache.org/) for service registry
* [Kubernetes](https://kubernetes.io/) for container orchestration

Currently, we only support Java language; however, we are planning to support Nodejs, Rust and Go in the future if there are enough customer's demands. For some of our customers, they have some existing RESTful APIs that built on top of other Java frameworks or other languages. We have frequently been asked on how to interact with these services to/from light services and how to enable security, metrics, logging, discovery, validation, sanitization, etc. on the existing services. 

Our answer is to deploy a reverse proxy built on top of the light-4j framework that wraps the existing service. 

The reverse proxy has the following features:

* High throughput, low latency and small footprint. 
* Integrate light-oauth2 to protect un-secured services
* Built-in load balancer
* Can be started with Docker or standalone
* Support HTTP 2.0 protocol on both in/out connections
* TLS termination
* Allow adding other handlers. For example, TableauAuthHandler.
* Support REST, GraphQL and RPC style of APIs
* Centralized logging with ELK, traceabilityId and CorrelationId
* Collect client and service metrics into InfluxDB and view the dashboard on Grafana
* Service registry and discovery with Consul or Zookeeper
* Manage configuration with light-config-server

To learn how to use this proxy pleases refer to 

* [Getting Started][] to learn core concepts
* [Tutorial][] with step by step guide for RESTful proxy
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* [TableauAuthHandler][] to handle authentication and get token for Tableau server

  
[Getting Started]: /getting-started/light-proxy/
[Tutorial]: /tutorial/proxy/
[Configuration]: /service/proxy/configuration/
[Artifact]: /service/proxy/artifact/
[TableauAuthHandler]: /service/proxy/tableau/
