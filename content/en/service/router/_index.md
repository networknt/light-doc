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

All the services developed on top of light-4j frameworks support [client side service discovery](http://microservices.io/patterns/client-side-discovery.html), load balance and cluster natively. However, for some clients who cannot leverage our client module, we have provided the light-router, which moves the client modules on the network to allow the original client to consume services as it is today without considering how dynamic these services are in the cloud.

Another usage for the light-router is to act as a BFF(backend for frontend) for Single Page Applications or Mobile Native Applications. It is supposed to be used only for simple use cases that the client wants to access backend APIs built on top of Light Platform. Suppose there is additional logic in the business context, for example, transformation, orchestration, aggregation and integration. In that case, you need to create customized middleware handlers and wire them into the light-router request/response chain or create a microservice for the BFF using light-router as a template. 

The third use case for the light-router is to act as a distributed gateway for an external client. The light-router will be responsible for security, load balance, service discovery and route traffic to the right services. We call this component EAP (External Access Point). When exposing internal microservices to an external client on the Internet, we can deploy an instance of light-router per external client in the B2B scenario. We call this component EAP (External Access Point). For organizations with limited resources and network limitations, a single instance of the light-router can be deployed for all external clients. 

Along with service discovery and routing, light-router also provides other cross-cutting concerns like security, metrics, logging, validation and more.

The light-router has the following features:

* High throughput, low latency, and small memory footprint. 
* TLS passthrough or termination
* Discovers service with Light-Portal Controller, Consul, etc.
* Supports environment tags for multiple testing environments deployed to the same cloud.
* Integrates light-oauth2, oauth-kafka and third-party OAuth 2.0 providers to retrieve the access token and renew the token before expiration.
* Built-in load balancer and cluster support
* Can be started with Docker or standalone
* Supports HTTP/2.0 protocol on both in/out connections
* Supports REST, GraphQL and RPC style of APIs
* Centralized logging to ELK with traceabilityId, and correlationId or Open Tracing
* Collects client and service metrics into InfluxDB and view the dashboard on Grafana
* Manages configuration with light-portal light-config-server 

To learn how to use this router, please refer to 

* [Getting Started][] to learn why we have provided this router
* [Tutorial][] with step by step guide for the RESTful router
* [Components][] router specific components and middleware handlers
* [Artifact][] find release artifacts and deploy options
* [Configuration][] for different configurations based on your situations
* [Location and Ownership] for deployment and ownership decision
* [Static Content][] render a single page application
* [Servic Id or URL] directives in header for downstream API invocation
* [URL Rewrite Rules] for light-router used as an API Gateway


[Getting Started]: /getting-started/light-router/
[Tutorial]: /tutorial/router/
[Components]: /service/router/components/
[Configuration]: /service/router/configuration/
[Artifact]: /service/router/artifact/
[Location and Ownership]: /service/router/location-ownership/
[Static Content]: /service/router/static-content/
