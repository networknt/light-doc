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

The Kafka sidecar is designed to address the following concerns for distributed microservices to leverage asynchronous event-based communications instead of synchronous request/response over HTTP. 

* Hide the complicity of Kafka client

Unlike traditional server-heavy messaging systems, Kafka's server is just a set of appended logs; however, the Kafka client is much more complicated. It is not an easy task to build a producer or consumer in microservices within a multi-threaded context. 

* Address asynchronous cross-cutting concerns

The sidecar will have middleware handlers in the request/response chain to address cross-cutting concerns, just like the light-proxy sidecar for the backend APIs. For example, the correlationId or tracerId can be injected into the message headers with the sidecar producer to propagate to downstream APIs with the sidecar consumer. 

* Easy to upgrade for new features and security vulnerabilities

There are many dependencies for Kafka and Confluent components on the client-side, and there are upgraded frequently to match the server-side version upgrade. Instead of updating all microservices to leverage the new server-side features or resolve security vulnerabilities, we can upgrade the sidecar and deploy it. 

* Support multiple languages and frameworks for APIs

Kafka client is complicated, and only the Java version is in sync with the server upgrades. The other best version that Confluent Inc. supports is the C++ version, and a lot of languages depend on it with a wrapper. There are several open-source clients for some languages, but they are always behind the server upgrades with minimum supports. With the sidecar, developers can choose any language or framework to build their APIs.
 
The goal is to allow the developers to focus on their APIs' business logic instead of the specific Kafka client API and cross-cutting concerns.

## Functional Component

### Kafka Producer

[Kafka Producer][] has an endpoint that allows backend API/APP to invoke to produce messages to one or more Kafka topics. 

### Active Consumer

[Kafka Active Consumer] has some endpoints to allow backend API/APP to create an instance, subscribe and retrieve records actively. When using this consumer, the backend API/APP has complete control of consuming the Kafka topic(s).

### Reactive Consumer

[Kafka Reactive Consumer][] will automatically poll the Kafka topic(s) on the sidecar and invoke an API endpoint on the back-end API/APP once messages are available based on the configuration. It has several endpoints to allow the light-controller to seek the offset if necessary. 

### KsqlDB Consumer

[Kafka KsqlDB Consumer][] will automatically subscribe from the KsqlDB server with a query during the server startup and callback to the backend API/App when records are received. 

### Kafka Streams

[Kafka Streams][] allows the users to develop and deploy the stream processor on the Kafka sidecar with the backend API exposes endpoints to receive the notification or the processed result. This component is working in progress. 

## Control Pane Administration

When using the Kafka Producer and Active Consumer, the backend API has full control over the producer and consumer; however, other consumers and streams are working reactively on the sidecar. The backend API will passively receive the post requests from the sidecar. If we want to update the consumer states for each consumer instance, we can leverage the control pane to access the sidecar's admin API. For example, if we want to replay events from a specific offset. For more info, please visit [Kafka Sidecar Admin][]

## Cross-cutting Concerns

One of the main goals for the Kafka sidecar is to address [cross-cutting concerns][] for asynchronous events consistently. The following non-functional concerns are address in the sidecar. 

* HTTP Request validation with OpenAPI 3.0 specification and schema validation and serialization with Confluent schema registry. 
* Propagate light-4j correlationId/traceabilityId or open tracing tracerId to the downstream service through Kafka record headers. 
* Inject correlationId/traceabilityId/tracerId to the MDC for logging statements and update logging level for sidecar packages from the control pane during runtime.
* Metrics collection on both client and service perspectives and integration with InfluxDB or enterprise metrics tools.
* The sidecar connects to the Kafka with authentication and authorization over TLS and only allow the connection from the same pod with localhost for HTTP requests.
* Sidecar auditing activities are sent to a Kafka topic through the producer and subsequently can be streams to a database for query with Confluent connect.
* Dead letter queue to handle the situation that the backend API cannot process the corrupted record successfully due to runtime exceptions. The problematic records will be pushed to a dead letter topic for future processing once the backend API is fixed and redeployed. 


[Kafka Active Consumer]: /service/mesh/kafka/active-consumer/
[Kafka Reactive Consumer]: /service/mesh/kafka/reactive-consumer/
[Kafka Producer]: /service/mesh/kafka/producer/
[Kafka KsqlDB Consumer]: /service/mesh/kafka/ksqldb-consumer/
[Kafka Streams]: /service/mesh/kafka/kafka-streams/
[Kafka Sidecar Admin]: /service/mesh/kafka/sidecar-amdin/
[cross-cutting concerns]: /service/mesh/kafka/sidecar-ccc/
