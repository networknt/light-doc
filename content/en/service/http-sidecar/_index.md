---
title: "HTTP Sidecar"
date: 2021-11-02T16:04:46-04:00
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

The [http-sidecar](/service/http-sidecar/) is a combination of [light-proxy](/service/proxy/) and [light-router](/service/router/) to manage the [cross-cutting concerns](/concern/) for the incoming and outgoing traffic of backend API in the same pod in a Kubernetes cluster.

- Package and deployed as a separate module to handle Cross-Cutting Concerns for the main container/service in the same pod. In this case, the main service only need care about the HTTP request/response and business logic

- Ingress traffic: First, client API requests will come to the sidecar service. Sidecar service acts as a proxy to delegate light client features, including open API schema validation, observability, monitoring, logging, JWT verify, etc. Then forward the request to the main service.

- Egress traffic: The main service calls the sidecar service first for egress traffic; in this case, the sidecar service acts as a router to delegate light client features, including service discovery, SSL handshake, JWT token management, etc. Then forward the request to server API. Here are several [egress configuration][] properties that need to be considered. 

Suppose an organization has made the decision to standardize with the sidecar approach. In that case, we highly recommend light-4j frameworks for backend API implementation for smooth integration with the HTTP Sidecar if Java 8 or Java 11 is the target language. However, users can build the backend API with any language and framework as the sidecar is deployed independently. 

You can deploy the HTTP Sidecar in the same container as the backend API or a separate container. We recommend the separate container approach based on the analysis in the [deploy patterns](/service/http-sidecar/deploy-patterns/). 

When the http-sidecar is used, all the traffic to and from the pod should go through the sidecar, and the [network policy][] will be defined to control the access. 

There are some special considerations for the [configurations](/service/http-sidecar/k8s-config/) for the http-sidecar when deploying it with a backend API in the same pod in a Kubernetes cluster. 

For most enterprise applications, sensitive data in the logs should be masked. It is easier to do that in the http-sidecar as a mask module is included. However, the masking must be handled differently for the backend API implemented in other Java frameworks or other languages. To support masking for the backend logs, we can send the backend logs to the http-sidecar, and http-sidecar can do the [log masking][] before injecting the log files to the target indexing app or pushing the logs to a Kafka cluster for streams processing. 

We need to deploy a multiple-container pod using the sidecar in a Kubernetes cluster. It will become more complicated when replicas, sealed secrets, network policies, auto scalers are involved. Instead of using the Helm for templating or Kustomize for patching, we leverage the light-config-server to manage the templates and light-bot to do the [Kubernetes deployment][]. 


[cross-cutting concerns]: /concern/
[network policy]: /service/http-sidecar/network-policy/
[log masking]: /service/http-sidecar/log-masking/
[Kubernetes deployment]: /service/k8s-deployment/
[egress configuration]: /service/egress-configuration/
