---
title: "Scheduler Local Development"
date: 2021-08-31T18:00:16-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The following are steps to get the development environment ready on your desktop. 


### Confluent Platform

You can start the Confluent Platform with [confluent local][] or [confluent docker][], and most users prefer the latter. 

https://youtu.be/L5RoJvCbkT0


### Create Topics

We need to create at least two topics for the task definitions and the other for the scheduled tasks. By default, we are using the light-scheduler topic name. Create the following two topics with 5 partitions from the [control center](http://localhost:9021).

```
light-scheduler
controller-health-check
```

### Configuration

The default configuration files in the src/resources/config folder are for enterprise Kafka cluster connection with all the security properties. For local development, we have simplified kafka-producer.yml, kafka-streams.yml and values.yml in the config/local folder. 

To run the light-scheduler server within the IDE or from the command line with local confluent. You need to set the JVM option to specify the config folder. 

```
-Dlight-4j-config-dir=config/local
```

For example, you can start the server from the terminal. 

```
cd ~/networknt/light-scheduler
mvn clean install -Prelease
java -Dlight-4j-config-dir=config/local -jar target/light-scheduler.jar

```

### Example request

Request to create a task definition for health check every 10 seconds.

```
curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
    "host": "networknt.com",
    "name": "market-192.168.1.2-health-check",
    "action": "INSERT",
    "frequency": {
      "timeUnit": "SECONDS",
      "time": 10
    },
    "topic": "controller-health-check",
    "data": {
      "key1": "value1",
      "key2": "value2"
    }
}'

```

Request to update the frequency to every 2 minutes.

```
curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
    "host": "networknt.com",
    "name": "market-192.168.1.2-health-check",
    "action": "UPDATE",
    "frequency": {
      "timeUnit": "MINUTES",
      "time": 2
    },
    "topic": "controller-health-check",
    "data": {
      "key1": "value1",
      "key2": "value2"
    }
}'

```

Request to cancel the task above with DELETE action.

```

curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
    "host": "networknt.com",
    "name": "market-192.168.1.2-health-check",
    "action": "DELETE",
    "frequency": {
      "timeUnit": "MINUTES",
      "time": 2
    },
    "topic": "controller-health-check",
    "data": {
      "key1": "value1",
      "key2": "value2"
    }
}'

```

Query active task definitions.

```
curl -k --location --request GET 'https://localhost:8401/schedulers'
```

### Code Walkthrough

{{< youtube pmPYqaQ-HIE >}}


[confluent local]: /tutorial/kafka-sidecar/confluent-local/
[confluent docker]: /tutorial/kafka-sidecar/confluent-docker/


