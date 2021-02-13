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

All the services developed on top of Light-4J frameworks support [client side service discovery](http://microservices.io/patterns/client-side-discovery.html), load balance, and cluster natively. So there is no need to put a reverse proxy instance in front of our services like other API frameworks that support only [server side service discovery](http://microservices.io/patterns/server-side-discovery.html). 

Light services also embed a distributed gateway to address all the cross-cutting concerns in the request/response chain and work with the ecosystem that consists:

* [light-oauth2](https://github.com/networknt/light-oauth2) for security
* [light-portal](https://github.com/networknt/light-portal) for API management and marketplace
* [light-config-server](https://github.com/networknt/light-config-server) for centralized configuration management
* [light-kafka](https://github.com/networknt/light-kafka) for eventual consistency based on event sourcing, CQRS and Kafka
* [ELK](https://www.elastic.co/webinars/introduction-elk-stack) for centralized logging with traceabilityId and correlationId
* [Prometheus](https://prometheus.io/), [InfluxDB](https://github.com/influxdata/influxdb) and [Grafana](https://github.com/grafana/grafana) for metrics
* [OpenTracing](https://opentracing.io/) with [Jaeger](https://www.jaegertracing.io/) tracer for distributed tracing and observability
* Light-Controller as part of the light-portal or [Consul](https://github.com/hashicorp/consul) for service registry
* [Kubernetes](https://kubernetes.io/) for container orchestration

Currently, we only support Java language; however, we are planning to support Nodejs, Rust, and Go in the future if there are enough customer's demands. Some of our customers have some existing RESTful APIs built on top of other Java frameworks or other languages. We have frequently been asked how to interact with these services to/from light services and enable security, metrics, logging, tracing, discovery, validation, sanitization, etc. on the existing services. 

Our answer is to deploy a reverse proxy built on top of the light-4j framework that wraps the existing service. 

The reverse proxy has the following features:

* High throughput, low latency, and small footprint. 
* Integrate light-oauth2 to protect un-secured services
* Built-in load balancer with multiple data centers and clouds support
* Can be started with Docker or standalone
* Support HTTP 2.0 protocol on both in/out connections
* TLS termination
* Allow adding other handlers—for example, TableauAuthHandler.
* Support REST, GraphQL and RPC style of APIs
* Centralized logging with ELK, TraceabilityId, and CorrelationId
* OpenTracing with Jaeger tracer integration
* Collect client and service metrics into InfluxDB or Prometheus and view the dashboard on Grafana
* Service registry and discovery with Portal Registry or Consul
* Manage configuration with light-config-server

To learn how to use this proxy, please refer to 

* [Getting Started][] to learn core concepts
* [Tutorial][] with step by step guide for RESTful proxy
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* [Body Validation][] skip the body validation on the proxy for better performance
* [TableauAuthHandler][] to handle authentication and get token for Tableau server
* [Proxy as a Sidecar][] to handle the cross-cutting concerns at network level
* [Proxy Benchmark][] to give confidence for users to leverage the proxy.

[Getting Started]: /getting-started/light-proxy/
[Tutorial]: /tutorial/proxy/
[Configuration]: /service/proxy/configuration/
[Artifact]: /service/proxy/artifact/
[TableauAuthHandler]: /service/proxy/tableau/
[Body Validation]: /service/proxy/body-validation/
[Proxy as a Sidecar]: /service/proxy/sidecar/
[Proxy Benchmark]: /service/proxy/benchmark/
