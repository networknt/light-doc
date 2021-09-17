---
title: "Kafka Mesh"
date: 2021-04-05T11:47:59-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Kafka Sidecar is part of the service mesh platform to handle all the cross-cutting concerns to work with the Kafka Cluster. 

### Producer

### ActiveConsumer

### CallbackConsumer



### KsqldbConsumer

The [KsqldbConsumer][] is part of the Kafka sidecar that subscribes to the KsqlDB server and calls back to the backend API. 

The [Ksqldb Active Consumer][] provide an endpoint to run the pull query against KsqlDB server and return the result as API response.


[KsqldbConsumer]: /tutorial/mesh/kafka/ksqldb/

[Ksqldb Active Consumer]: /tutorial/mesh/kafka/ksqldb_active/
