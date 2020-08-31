---
title: "Rest with Kafka Access"
date: 2020-08-30T23:38:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We used to leverage database for Event Sourcing and CQRS with light-eventuate-4, light-tram-4j and light-saga-4j. Due to multiple dependencies, it is hard to use. With the latest Kafka supports transactions, we have developed Light-Kafka to use Kafka Streams for Event Sourcing and CQRS. 

When using light-kafka with Event Sourcing, we don't put producer endpoints and consumer endpoints in the same service. For the rest of the document, we will call the producer as the command service and consumer as the query service. 

### Command Service

The command service is the producer of the Kafka streams. It exposes REST endpoints to the outside to accept data, validate and push to a Kafka topic. The format of the data normally will be JSON or AVRO. JSON is acceptable; however, we highly recommend AVRO for any enterprise application if backward and forward compatibility is required. 

### Query Service

