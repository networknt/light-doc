---
title: "SideCar"
date: 2020-11-24T10:25:43-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

One of the main features for light-4j frameworks is separating the cross-cutting concerns from the business logic so that developers can focus on the business API handler implementations and DevOps can focus on the cross-cutting concerns in the pipeline for deployment. 

All cross-cutting concerns are implemented as middleware handlers that can be plugged into the HTTP request/response chain. All of them group together to form a flexible gateway to protect the API handler implementations. 

These plugins can be embedded in the same process in the API implementation as an embedded gateway. However, this might limit API implementation on the light-4j frameworks. 

![embedded](/images/embedded-chain.png)


To support other frameworks and languages, we can deploy the light-proxy container as a sidecar along with API implementation. 


![proxy](/images/sidecar-chain.png)


When using the light-proxy as a sidecar in a Kubernetes pod, the light-proxy is acting as a micro gateway. It is not only a reverse proxy to address all the HTTP cross-cutting concerns, but also it can help with cross-cutting concerns for other system integration. 

For example, by leveraging the handlers in light-kafka on the proxy instance, we can translate the Kafka protocol to HTTP to interact with Kafka with REST API for backend service. Simultaneously, the proxy will address the messaging level of cross-cutting concerns for the producer and consumer. 

Another example is the mainframe integration. Instead of each API to handle the IMS connect, we can let the proxy do the translation, address common cross-cutting concerns and expose HTTP API to the back-end service to send requests.

The following diagram describes the usage. 

![kafka](/images/light-4j-sidecar.png)

This configuration ensures that the cross-cutting concerns are standardized in the organization. For all the Kafka related cross-cutting concerns, please visit [light-kafka](/style/light-kafka/). 

For example, if the business API wants to send a message to a Kafka topic, it can call the sidecar KafkaProduer REST endpoint with a topic, key, and value as the request body. The sidecar will send the key/value to the Kafka topic and at the same time inject cross-cutting concerns ProducerRecord headers.

The advantage of using the sidecar is that the backend API can be implemented in any framework and language. The developers don't need to have the knowledge to work with Kafka and don't need to worry about the cross-cutting concerns. 

The disadvantage of using the sidecar is that the business API won't fully control the Kafka produce and consumer. In the case that business API wants to have full control, it is recommended to implement with Light-Kafka. 

