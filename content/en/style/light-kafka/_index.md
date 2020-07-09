---
title: "Light Kafka"
date: 2020-07-09T14:46:19-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When deploying microservices to the cloud, there are two different ways for service to service communication within an organization. Most people are familiar with the request/response style of communication like REST, GraphQL or RPC. However, more organizations are adopting event-based communication with Event Sourcing and CQRS to ensure data consistency across services. We recently decommissioned the light-eventuate-4j, light-tram-4j and light-saga-4j and replaced them with light-kafka for Event Sourcing and CQRS framework. 

The latest Kafka supports transaction, so that we don't need an external database for event store. This significantly reduce the complicity. Also, the newly introduced Kafka streams is perfect fit to project the raw events in a topic to material views in different demensions. 

The latest Kafka supports transactions so that we don't need an external database for the event store. This significantly reduces the complexity of the system and simplify the dependencies. Also, the newly introduced Kafka streams is a perfect fit to project the raw events in a topic to material views in different dimensions. 

The light-kafka consists of four modules:

* kafka-common contains all the shared classes
* kafka-producer is used to construct and send events to a Kafka topic
* kafka-consumer is used to consume an event from a Kafka topic
* kafka-streams is used to process the event in a topic in real-time

If you are interested in using this module to develop event-based microservices, please contact sales@lightapi.net for details. 

