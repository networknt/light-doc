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

Please be aware we are using -Prelease to build the fatjar so that all the dependencies will be packaged into the jar. Now the server should be started with HTTPS/2 on port 8443. You can also start the server within the IDE to debug your Kafka Streams application. 


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

In the next step, we will drop the jar file to Kafka-sidecar and let the sidecar to host the Kafka Streams application. 

### Kafka Sidecar

In the kafka-sidecar repository, we have a folder under config named streams. Within this folder, we have config files and values.yml to support the above streams application. 

Here is the values.yml file that we have modified for the configuration of the streams application. We use docker-compose service names for the bootstrap servers and the registry URL. 

```
# streams configuration
kafka-streams.application.id: word-count-docker
kafka-streams.bootstrap.servers: broker:29092
kafka-streams.schema.registry.url: http://schema-registry:8081

```

For startup hooks and shutdown hooks, we add the classes from the example application. 

```
# Service Startup and Shutdown Hooks
service.com.networknt.server.StartupHookProvider:
  # - com.networknt.mesh.kafka.ProducerStartupHook
  - com.networknt.kafka.WordCountStartupHook
  # - com.networknt.mesh.kafka.ActiveConsumerStartupHook
  # - com.networknt.mesh.kafka.KsqldbReactiveConsumerStartupHook
  # - com.networknt.mesh.kafka.KsqldbActiveConsumerStartupHook
  # - com.networknt.mesh.kafka.ReactiveConsumerStartupHook
  # - com.networknt.mesh.kafka.AdminClientStartupHook
service.com.networknt.server.ShutdownHookProvider:
  # - com.networknt.mesh.kafka.ProducerShutdownHook
  - com.networknt.kafka.WordCountShutdownHook
  # - com.networknt.mesh.kafka.ActiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.KsqldbReactiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.KsqldbActiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.ReactiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.AdminClientShutdownHook

```

To make the application run within the Kafka-sidecar, we must copy the streams application jar file to the streamsjar folder in the kafka-sidecar repository. Note: we don't need to copy the fatjar as the Kafka-sidecar contains all the dependencies to run a Kafka Streams application. 

Since we have built the application with fatjar before with `mvn clean install -Prelease`, we just need to copy the original jar file. 

```
cd ~/networknt/kafka-sidecar
cp ~/networknt/light-example-4j/kafka-sidecar/kafka-streams-dsl/target/original-word-count-1.0.0.jar ./streamsjar
```

Note that the original jar is used as the Kafka-sidecar will provide all the dependencies. Now, we can start the kafka-sidecar with the following docker-compose.

```
cd ~/networknt/kafka-sidecar
docker-compose -f docker-compose-streams.yml up
```

Once the server is up, we can test with the existing console producer and consumer. Type some sentences on the producer console; you should see the word count changing on the consumer console. 

### Next

We can enable the producer on the sidecar so that we can produce to the input topic with the producer API instead of the console producer. 

We can add a backend consumer to consumer the messages from the output topic and output from the docker-compose console to replace the console consumer. 

We can create another Kafka-streams application and drop it into the same streamsjar folder to start two streams applications on the same sidecar instance.  


[DSL]: https://github.com/networknt/light-example-4j/tree/master/kafka-sidecar/kafka-streams-dsl
