---
title: "Light Kafka"
date: 2020-11-28T15:34:56-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When deploying large scale microservices architecture, it is crucial to mix synchronous request/response with asynchronous message/command/event over message bus like Kafka. Each service will have some endpoints to serve the consumer's calls over the HTTP and maintain the data consistency with other services as each service is managing the data independently. 

Most of our customers are using Kafka for the message broker, and we have developed light-kafka to help our users integrate and address the cross-cutting concerns. Like the middleware handlers in the light-4j project, we have developed a group of middleware plugins specifically for the messaging system. 


### Modules

In the light-kafka repository, we have four modules: 

##### kafka-common

This module contains Message, Command, Event interfaces. Configurations files for the producer, consumer and streams as well as the De/Serializer. 

##### kafka-producer

This module provides LightProducer interface and the TransactionalProducer to push records into a Kafka topic in batch and transaction. 

##### kafka-consumer

This module provides LightConsumer interface for microservice to implement in the startup hook. 

##### kafka-streams

This module provides LightStreams interface for microservice to implement in the startup hook.

### Cross-Cutting Concerns

When we have HTTP service invocations in a chain, we propagate the JWT token, traceabilityId, correlationId, tracer, etc., in the HTTP headers to the entire chain of the services with Http2Client. It will allow the logging, metrics, tracing to be linked together for a particular request in the centralized location. However, if there is an asynchronous invocation in the middleware of the chain, the HTTP headers will be lost in the middle. To close the loop, we decide to inject the HTTP headers to the Kafka ProducerRecord headers from a service implementing the LightProducer interface and retrieves the headers in service either implements LightConsumer or LightStreams interface. The LightProducer service will pick up the HTTP headers and put them into the ProducerRecord before sending the message/command/event to a Kafka topic. The LightConsumer or LightStreams service will pick up the headers from the message/command/event and put them into the HTTP headers to the next Kafka Producer record (async) or HTTP headers (sync) to propagate them in the chain.
 
##### Security

For a LightStreams or LightConsumer to process a message/command/event from a Kafka topic, it doesn't know who sent it and if the LightProducer has authorized to do so. Passing the JWT tokens in the ProducerRecord header, a downstream API can identify who is the original caller and which service is the LightProducer. Given the reliability of the LightConsumer or LightStreams along with the speed of the Kafka cluster, we might need to give the JWT token more grace period or even avoid validation on the expiration. The same JWT tokens can provide input for the subsequent metrics and audit to provide the callerId and clientId. 

##### Validation

Depending on JSON or Avro's message/command/event format, we have two different validation approaches. If Avro is used, the validation is built-in, and the Schema Registry of the Confluent is used to ensure backward and forward compatibility. When JSON is used, we can leverage the networknt [json-schema-validator](https://github.com/networknt/json-schema-validator) to validate on the LightConsumer and LightStreams implementation. 

##### Tracing

Light-4j provides two options for tracing. The internal correlationId/traceabilityId and Open Tracing with Jaeger and SkyWalking. All the tracing ids are propagated in the HTTP headers and Kafka ProducerRecord headers. 


##### Metrics

Metrics has two dimensions; the service view captures the information based on the serviceId, and the client view needs the clientId to be in the JWT token. When the JWT token is propagated from the Kafka ProducerRecord headers, the clientId can be passed to the subsequent services when metrics are collected. 


##### Audit

Like the metrics, the Audit handler needs some information from the HTTP headers or Kafka ProducerRecord headers. For example, the clientId is from the JWT token that is propagated in the invocation chain.




