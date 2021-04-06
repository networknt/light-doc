---
title: "Kafka Sidecar"
date: 2021-03-16T14:25:35-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Unlike other messaging systems with a heavy server and light client, the Kafka server is relatively simple as it is an append log, but the client is very complicated. To hide all the complexities, developers can access the Kafka cluster with a sidecar in the same pod. 

The sidecar will have middleware handlers in the request/response chain to address cross-cutting concerns, just like the light-proxy sidecar for the backend APIs. 

### Kafka Producer

[Kafka Producer][] has an endpoint that allows backend API/APP to invoke to produce messages to one or more Kafka topics. 

### Active Consumer

[Kafka Active Consumer] has some endpoints to allow backend API/APP to create an instance, subscribe and retrieve records actively. When using this consumer, the backend API/APP has complete control of consuming the Kafka topic(s).

### Callback Consumer

[Kafka Callback Consumer][] will automatically poll the Kafka topic(s) on the sidecar and invoke an API endpoint on the back-end API/APP once messages are available based on the configuration. It has several endpoints to allow the light-controller to seek the offset if necessary. 

### KsqlDB Consumer

[Kafka KsqlDB Consumer][] will automatically subscribe from the KsqlDB server with a query during the server startup and callback to the backend API/App when records are received. 

[Kafka Active Consumer]: /service/mesh/kafka/active-consumer/
[Kafka Callback Consumer]: /service/mesh/kafka/callback-consumer/
[Kafka Producer]: /service/mesh/kafka/producer/
[Kafka KsqlDB Consumer]: /service/mesh/kafka/ksqldb-consumer/
