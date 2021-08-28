---
title: "Local Binary"
date: 2021-08-28T00:08:01-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


### Check Out

Check out the following repositories to your local working directory. I am using networknt in my home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-kafka.git
git clone https://github.com/networknt/light-example-4j.git

```


### Confluent Local

Before running the tutorial, we need to have confluent platform started locally. If you are using Linux, you can install confluent locally and start it from the command line. 

```
cd ~/tool/confluent-6.1.0/bin
confluent local services start

```

You can also start the confluent locally with a docker-compose. 

```
cd ~/networknt/light-kafka
docker-compose up

```

Once the confluent platform is started, go to http://localhost:9021 to access the control center. Create a topic named test6 without key schema and value schema. 


{{< youtube A00tGBNayCk >}}


### Start Backend

Let's first start the ReactiveConsumer backend service within the IDE from the following folder. We will start it from the command line later.

```
~/networknt/light-example-4j/kafka/sidecar-backend

```

### Start Sidecar

Start the Kafka sidecar within the IDE from the following folder.

```
~/networknt/kafka-sidecar

```

set the JVM Option to

```
-Dlight-4j-config-dir=config/local
```

{{< youtube tnvKQuXRzEc >}}


### Binary Value

update the config/local/values.yml to the following. 

```
# producer format configuration
kafka-producer.keyFormat: string
kafka-producer.valueFormat: binary

# consumer format configuration
kafka-consumer.topic: test6
kafka-consumer.keyFormat: string
kafka-consumer.valueFormat: binary

# Service Startup and Shutdown Hooks
service.com.networknt.server.StartupHookProvider:
  - com.networknt.mesh.kafka.ProducerStartupHook
  # - com.networknt.mesh.kafka.ActiveConsumerStartupHook
  # - com.networknt.mesh.kafka.KsqldbConsumerStartupHook
  - com.networknt.mesh.kafka.ReactiveConsumerStartupHook
  # - com.networknt.mesh.kafka.AdminClientStartupHook
service.com.networknt.server.ShutdownHookProvider:
  - com.networknt.mesh.kafka.ProducerShutdownHook
  # - com.networknt.mesh.kafka.ActiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.KsqldbConsumerShutdownHook
  - com.networknt.mesh.kafka.ReactiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.AdminClientShutdownHook


```

Curl command line.

```
curl -k --location --request POST 'https://localhost:8443/producers/test6' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":"alice","value":"YWJj"},{"key":"john","value":"ZGVm"},{"key":"alex","value":"eHl6"}]}'

```

{{< youtube UBGNyGUgkys >}}



### Binary Key

Update the config/local/values.yml to the following.

```

# producer format configuration
kafka-producer.keyFormat: binary
kafka-producer.valueFormat: string

# consumer format configuration
kafka-consumer.topic: test6
kafka-consumer.keyFormat: binary
kafka-consumer.valueFormat: string

# Service Startup and Shutdown Hooks
service.com.networknt.server.StartupHookProvider:
  - com.networknt.mesh.kafka.ProducerStartupHook
  # - com.networknt.mesh.kafka.ActiveConsumerStartupHook
  # - com.networknt.mesh.kafka.KsqldbConsumerStartupHook
  - com.networknt.mesh.kafka.ReactiveConsumerStartupHook
  # - com.networknt.mesh.kafka.AdminClientStartupHook
service.com.networknt.server.ShutdownHookProvider:
  - com.networknt.mesh.kafka.ProducerShutdownHook
  # - com.networknt.mesh.kafka.ActiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.KsqldbConsumerShutdownHook
  - com.networknt.mesh.kafka.ReactiveConsumerShutdownHook
  # - com.networknt.mesh.kafka.AdminClientShutdownHook

```

{{< youtube YFoOc4Pou7g >}}


### Command Line


We are going to start both Kafka Sidecar and the sidecar-backend with the command line and run the curl command from the command line to test the sidecar. 


{{< youtube vTumPaWd9RU >}}


### Docker Compose

{{< youtube D0LNMtteF_0 >}}


### Kubernetes

