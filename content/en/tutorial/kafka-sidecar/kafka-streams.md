---
title: "Kafka Streams"
date: 2022-04-18T22:42:34-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Kafka-sidecar supports Kafka Streams Applications with drop-in support. You just need to drop your streams application jar file into the folder mapped to /streamsjar in Docker volume. 

### Start Kafka

Before running any demo applications, we need to get the Confluent Platform started. If you have Docker installed, you can start it from the kafka-sidecar repository. 

```
cd ~/networknt
git clone https://github.com/networknt/kafka-sidecar.git
cd kafka-sidecar
docker-compose down
docker-compose up
```

### Test Topics

Once the Confluent Platform is up and running, create two topics from the control center or with the following command line. 

```
cd ~/networknt/kafka-sidecar

docker-compose exec broker kafka-topics --create --if-not-exists --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic streams-plaintext-input --config retention.ms=-1
docker-compose exec broker kafka-topics --create --if-not-exists --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic streams-wordcount-output --config retention.ms=-1
```

### Create Jar

The first step is creating a Kafka streams application, testing it, and building a jar file. We will soon create a light-codegen generator to help users scaffold a new project. For now, you can choose an example application from light-example-4j/kafka-sidecar for [DSL][] or Processor API, depending on your business use cases. 


### Kafka Streams with DSL

If you want to start a Kafka Streams application with DSL, you can check out the light-example-4j and switch to the master branch. 


```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j
git checkout master
cd kafka-sidecar/kafka-streams-dsl
mvn clean install -Prelease
java -jar target/word-count-1.0.0.jar
```

Now the server should be started with HTTPS/2 on port 8443. You can also start the server within the IDE to debug your Kafka Streams application. 


### Test DSL Streams Application

First, let's start a console consumer to consume the messages from the streams-wordcount-output topic. 


```
cd ~/networknt/kafka-sidecar
docker-compose exec broker kafka-console-consumer --topic streams-wordcount-output --from-beginning --bootstrap-server localhost:9092 --property print.key=true --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

Second, let's start a console producer and input some words. 

```
cd ~/networknt/kafka-sidecar
docker-compose exec broker kafka-console-producer --broker-list localhost:9092 --topic streams-plaintext-input
```

After entering some words like below, we should see the result from the consumer console. 


Producer Console

```
>hello
>world
>hello
>world
>hello
>world
>world
>world
>world
>world
>world
>world
>
```

Consumer Console

```
hello	2
world	2
hello	3
world	3
world	5
world	9
```

The above example is a direct conversion of word count example from Confluent GitHub [repository](https://github.com/confluentinc/kafka-streams-examples/blob/7.0.0-post/src/main/java/io/confluent/examples/streams/WordCountLambdaExample.java). 

In the next step, we will implement the same example with Processor API.

### Kafka Streams with Processor API



[DSL]: https://github.com/networknt/light-example-4j/tree/master/kafka-sidecar/kafka-streams-dsl
