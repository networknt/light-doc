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

* [Light-Proxy](/service/proxy/)

The light-proxy can be used as a sidecar to address cross-cutting concerns for APIs built with any framework and language. 


* [Kafka-Sidecar](/service/mesh/kafka/)

Kafka sidecar contains both producer and consumers (consumer group, stream processor and ksqldb subscriber) to interact with a Kafka cluster without the developer knowing Kafka client. All the interactions to Kafka are through the sidecar with REST APIs. Like using the light-proxy as a sidecar, we can leverage light-4j middleware handlers to address cross-cutting concerns at the sidecar level and propagate tracer to the ProducerRecord headers.


