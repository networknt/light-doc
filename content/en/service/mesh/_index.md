---
title: "Light Mesh"
date: 2021-03-16T13:57:35-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 8
weight: 8	#rem
slug: ""
toc: false
draft: false
reviewed: true
---

Several sidecars in the light-mesh project can be used as a sidecar container in a Kubernetes pod along with your API built with any frameworks or languages to address cross-cutting concerns. With most of the cross-cutting concerns addressed, your developers can focus on the business logic only in the backend API. 

### What Is a Sidecar Pattern

Segregating the functionalities of an application into a separate process can be viewed as a Sidecar pattern. The sidecar design pattern allows you to add a number of capabilities to your application without additional configuration code for third-party components.

In software architecture a sidecar attach to a parent application and extends/enhances its functionalities. A sidecar is loosely coupled with the main application.

In Kubernetes cluster environment, sidecar can be deployed as Sidecar container run in parallel with the main container in the pod.



### Benefits of Using a Sidecar Pattern:



- Reduces the complexity in the microservice code by abstracting the common infrastructure-related functionalities to a different layer.


- Reduces code duplication in a microservice architecture since you do not need to write configuration code inside each microservice.


- Provide loose coupling between application code and the underlying platform.



* [Http-sidecar](/service/mesh/http/)

The Http-sidecar integrate light-proxy and light-router features to handler ingress and egress traffic in a Kubernetes pod;  and it can be used as a sidecar to address cross-cutting concerns for APIs built with any framework and language. 


* [Kafka-Sidecar](/service/mesh/kafka/)

Kafka sidecar contains both producer and consumers (consumer group, stream processor and ksqldb subscriber) to interact with a Kafka cluster without the developer knowing Kafka client. All the interactions to Kafka are through the sidecar with REST APIs. Like using the light-proxy as a sidecar, we can leverage light-4j middleware handlers to address cross-cutting concerns at the sidecar level and propagate tracer to the ProducerRecord headers.


