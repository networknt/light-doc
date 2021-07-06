---
title: "Schema Registry Installation"
date: 2018-11-30T16:54:36-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In most cases, we are using Open Source Kafka, Zookeeper and Confluent Schema Registry for our applications. For some customers, they have the enterprise license and support from Confluent Inc., but we have to focus on the access for most of the users. 

If you haven't had Kafka and ZooKeeper installed, please refer to the [installation document][]. 

[Schema registry][] is an open source server from Confluent, and it can be easily installed via Docker. 

Here is the docker command to start it on one of our VMs. 

```
docker run -d \
-e SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=38.113.162.53:2181 \
-e SCHEMA_REGISTRY_HOST_NAME=schema-registry \
-e SCHEMA_REGISTRY_LISTENERS=http://0.0.0.0:8081 \
confluentinc/cp-schema-registry:5.3.1
```

In a test environment, we only need one instance of the schema registry. 

[installation document]: /service/messaging/kafka/installation/
[Schema registry]: https://github.com/confluentinc/schema-registry/
