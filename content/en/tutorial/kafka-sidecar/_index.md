---
title: "Kafka Sidecar"
date: 2021-08-16T08:05:11-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---


As part of [light-mesh](/service/mesh/) offering, the [Kafka Sidecar](/service/mesh/kafka/) is an infrastructure component to help businesses to build services interacting with enterprise Kafka clusters. In this tutorial, I will outline the major steps to get the sidecar and backend API up and running with a local Confluent and up to production deployed to a Kubernetes cluster. 

* [Local development][] with [Confluent local][]
* [Local binary][] demo with externalized configuration
* [Reactive Consumer Producer] backend reference implementation to receive a batch of records, transform the key with a field in the value and produce the new key/value to another topic. 
* [Kafka Streams] with DSL or Processor API and deploy it with the Kafka-sidecar.


[Local development]: /tutorial/kafka-sidecar/local-dev/
[Local binary]: /tutorial/kafka-sidecar/local-binary/
[Confluent local]: /tutorial/kafka-sidecar/confluent-local/
[Confluent Docker]: /tutorial/kafka-sidecar/confluent-docker/
[Reactive Consumer Producer]: /tutorial/kafka-sidecar/reactive-consumer-producer/
[Kafka Streams]: /tutorial/kafka-sidecar/kafka-streams/
