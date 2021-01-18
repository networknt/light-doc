---
title: "Kafka Streams Report"
date: 2021-01-06T09:27:36-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In the previous [stream-query][] tutorial, we have demonstrated how to build a Kafka Streams service with light-rest-4j. This tutorial will create another microservice with Kafka Streams to consume the same topic with the event sourcing. In the real business environment, this service might be created several years later when the CEO asks how many registered users per country per year since the beginning. 

The project is the report service that has /report get endpoint. 

### Codegen

As usual, you can find the specification, config.json and a README.md with a light-codegen command line in the model-config repository.

https://github.com/networknt/model-config/tree/master/kafka/stream-report

Follow the README.md command lines, we can generate a project in the light-example-4j/kafka folder.

https://github.com/networknt/light-example-4j/tree/master/kafka/stream-report

The project can be built and started with the following command.

```
mvn clean install exec:exec
```

### pom.xml

We need to add the kafka-streams dependencies to the pom.xml

```
        <version.kafka>2.5.0</version.kafka>


        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-common</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>${version.kafka}</version>
        </dependency>

```



[stream-query]: /tutorial/light-kafka/stream-query/

