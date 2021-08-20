---
title: "HTTP Sidecar"
date: 2021-06-02T11:31:35-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

There are two ways to implement a complete distributed microservices application with Light in a Kubernetes cluster: Embedded and Sidecar. 

If you use Java 8 or Java 11 to implement your service, you can leverage one of the Light frameworks with [cross-cutting concerns][] embedded in the request/response chain. 

The other option is to use the HTTP Sidecar container in the same pod to address the [cross-cutting concerns][] at the network level. 

The HTTP Sidecar is a combination of [light-proxy](/service/proxy/) and [light-router](/service/router/) to manage the [cross-cutting concerns](/concern/) for the incoming and outgoing traffic of backend API in the same pod in a Kubernetes cluster.

- Package and deployed as separate module to handle Cross-Cutting Concerns for main container/service in the same pod. In this case, the main service only need care about the http request/response and business logic

- Ingress traffic: client API request will come to sidecar service first, sidecar service act as a proxy to delegate light client features, which include, openapi schema validation, observability, monitoring, logging, JWT verify, etc. Then forward the request to main service.

- Egress traffic: main service call sidecar service first for egress traffic; in this case, sidecar service act as a router to delegate light client features, which include service discovery, SSL handshake, JWT token management, etc. Then forward the request to server API.

Suppose an organization has made the decision to standardize with the sidecar approach. In that case, we highly recommend light-4j frameworks for backend API implementation for smooth integration with the HTTP Sidecar if Java 8 or Java 11 is the target language. However, users can build the backend API with any language and framework as the sidecar is deployed independently. 

You can deploy the HTTP Sidecar in the same container as the backend API or in a separate container. We recommend the separate container approach based on the analysis in the [deploy patterns](/service/mesh/http/deploy-patterns/). 

When http-sidecar is used, all the traffic to and from the pod should go through the sidecar, and the [network policy][] will be defined to control the access. 

There are some special consideration for the [configurations](/service/mesh/http/k8s-config/) for the http-sidecar when deploy it with a backend API in the same pod in a Kubernetes cluster. 

[cross-cutting concerns]: /concern/
[network policy]: /service/mesh/http/network-policy/
