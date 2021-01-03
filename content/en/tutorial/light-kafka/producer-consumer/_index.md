---
title: "Producer and Consumer"
date: 2021-01-02T21:28:52-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This tutorial will build a microservice that produces a list of key-value pairs to a Kafka topic and consumes the list of records with another get endpoint.

### Codegen

To scaffold a new project, we put the README.md, openapi.yaml and config.json to the model-config repo in the following folder.

https://github.com/networknt/model-config/tree/master/kafka/producer-consumer

The command line is in the README.md, and the specification has two endpoints. 

The generated project is located in the light-example-4j. 

https://github.com/networknt/light-example-4j/tree/master/kafka/producer-consumer

You can build and run the generated project from the command line with:

```
mvn clean install exec:exec
```

### Producer

First, let's implement the producer-handler. It accepts a list of Key/Value pairs and sends them to a Kafka topic specified by the path parameter. 

##### pom.xml

In the pom.xml, we add the following dependency. 

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-producer</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

##### ProducerStartupHook.java and ProducerShutdownHook.java

We need to create a startup hook to initialize the Kafka producer during the server startup and a shutdown hook to close the Kafka producer during the shutdown. 

ProducerStartupHook.java

```
package com.networknt.kafka;

import com.networknt.service.SingletonServiceFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.networknt.kafka.producer.LightProducer;
import com.networknt.kafka.producer.TransactionalProducer;
import com.networknt.server.StartupHookProvider;

public class ProducerStartupHook implements StartupHookProvider {
    private static Logger logger = LoggerFactory.getLogger(ProducerStartupHook.class);
    @Override
    public void onStartup() {
        logger.debug("ProducerStartupHook start");
        TransactionalProducer producer = (TransactionalProducer) SingletonServiceFactory.getBean(LightProducer.class);
        producer.open();
        new Thread(producer).start();
        logger.debug("ProducerStartupHook complete");
    }
}

```

ProducerShutdownHook.java

```
package com.networknt.kafka;

import com.networknt.kafka.producer.LightProducer;
import com.networknt.server.ShutdownHookProvider;
import com.networknt.service.SingletonServiceFactory;

public class ProducerShutdownHook implements ShutdownHookProvider {
    @Override
    public void onShutdown() {
        LightProducer producer = SingletonServiceFactory.getBean(LightProducer.class);
        try { if(producer != null) producer.close(); } catch(Exception e) {e.printStackTrace();}
    }
}

```

##### service.yml

We need to add the producer startup and shutdown hooks to the service.yml and pick a producer implementation optimized for throughput.

service.yml

```
# Singleton service factory configuration/IoC injection
singletons:
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.kafka.ProducerStartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.kafka.ProducerShutdownHook
- com.networknt.kafka.producer.LightProducer:
  - com.networknt.kafka.producer.TransactionalProducer

```

##### kafka-producer.yml

This is the configuration file for the light-kafka producer. 

```
---
acks: all
retries: 3
batchSize: 16384
lingerMs: 1
bufferMemory: 33554432
keySerializer: org.apache.kafka.common.serialization.ByteArraySerializer
valueSerializer: org.apache.kafka.common.serialization.ByteArraySerializer
keyDeSerializer: org.apache.kafka.common.serialization.ByteArraySerializer
valueDeSerializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
sessionTimeout: 30000
autoOffsetreset: earliest
enableAutoCommit: false
bootstrapServers: localhost:9092
# every server instance will have a unique transactionId
transactionId: T1000
transactionTimeoutMs: 900000
transactionalIdExpirationMs: 2073600000

```

##### ProducerTopicPostHandler.java

Update the generated handler to get the topic and a list of key/value pairs to send to Kafka. 

```
package com.networknt.kafka.handler;

import com.networknt.body.BodyHandler;
import com.networknt.handler.LightHttpHandler;
import com.networknt.kafka.producer.LightProducer;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.server.HttpServerExchange;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.Map;
import java.util.concurrent.BlockingQueue;

public class ProducerTopicPostHandler implements LightHttpHandler {
    private static Logger logger = LoggerFactory.getLogger(ProducerTopicPostHandler.class);
    private static String STATUS_ACCEPTED = "SUC10202";
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        logger.debug("ProducerTopicPostHandler start");
        String topic = exchange.getQueryParameters().get("topic").getFirst();
        logger.info("topic: " + topic);
        List<Map<String, Object>> list = (List)exchange.getAttachment(BodyHandler.REQUEST_BODY);

        LightProducer producer = SingletonServiceFactory.getBean(LightProducer.class);
        BlockingQueue<ProducerRecord<byte[], byte[]>> txQueue = producer.getTxQueue();

        for (int i = 0; i < list.size(); i++) {
            Map<String, Object> itemMap = list.get(i);
            String key = (String)itemMap.get("key");
            String value = (String)itemMap.get("value");
            ProducerRecord<byte[], byte[]> record = new ProducerRecord<>(topic, key.getBytes(StandardCharsets.UTF_8), value.getBytes(StandardCharsets.UTF_8));
            try {
                txQueue.put(record);
            } catch (InterruptedException e) {
                logger.error("Exception:", e);
            }
            logger.debug("Send message with key = " + key + " value = " + value  + " to topic: " + topic);
        }
        setExchangeStatus(exchange, STATUS_ACCEPTED);
    }
}

```

##### Test Producer

We need to start the confluent locally and create a test topic. 

```
confluent local start

kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic test --config retention.ms=-1
```

To consume the message from test topic on the console. 

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --property print.key=true --property key.separator="-" --from-beginning
```

To start the server.

```
mvn clean install exec:exec
```

Send a request from curl and make sure application/json content-type is used.

```
curl -k --location --request POST 'https://localhost:8443/producer/test' \
--header 'Content-Type: application/json' \
--data-raw '[{"key":"key1","value":"value1"},{"key":"key2","value":"value2"}]'

```

From the consumer console, you should see the following lines. 

```
key1-value1
key2-value2
```

### Consumer

In the above producer test, we use the Kafka console consumer to verify that we have messages pushed to the Kafka topic from our producer-handler. In this step, we are going to implement a consumer handler. 


##### pom.xml

Add the following dependency

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>kafka-consumer</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

##### ConsumerStartupHook.java and ConsumerShutdownHook.java

We need to create a startup hook to initialize the Kafka consumer during the server startup and a shutdown hook to close the Kafka consumer during the shutdown. 

ConsumerStartupHook.java

```
package com.networknt.kafka;

import com.networknt.kafka.consumer.LightConsumer;
import com.networknt.server.StartupHookProvider;
import com.networknt.service.SingletonServiceFactory;

public class ConsumerStartupHook implements StartupHookProvider {
    public static KeyValueConsumer consumer;
    @Override
    public void onStartup() {
        consumer = (KeyValueConsumer) SingletonServiceFactory.getBean(LightConsumer.class);
        consumer.open();
    }
}

```

ConsumerShutdownHook.java


```
package com.networknt.kafka;

import com.networknt.kafka.consumer.LightConsumer;
import com.networknt.server.ShutdownHookProvider;
import com.networknt.service.SingletonServiceFactory;

public class ConsumerShutdownHook implements ShutdownHookProvider {
    @Override
    public void onShutdown() {
        LightConsumer consumer = SingletonServiceFactory.getBean(LightConsumer.class);
        try { if(consumer != null) consumer.close(); } catch(Exception e) {e.printStackTrace();}
    }
}

```

##### KeyValueConsumer.java

This is the consumer implementation that extends from the AbstractConsumer in the light-kafka. 

```
package com.networknt.kafka;

import com.networknt.kafka.consumer.AbstractConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;

import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.*;

public class KeyValueConsumer extends AbstractConsumer {
    public void subscribe(String topic) {
        consumer.subscribe(Arrays.asList(topic));
    }

    public List<Map<String, Object>> poll() {
        List<Map<String, Object>> list = new ArrayList<>();
        ConsumerRecords<byte[], byte[]> records = consumer.poll(Duration.ofMillis(100));
        for(ConsumerRecord<byte[], byte[]> record: records) {
            String key = new String(record.key(), StandardCharsets.UTF_8);
            String value = new String(record.value(), StandardCharsets.UTF_8);
            Map<String, Object> map = new HashMap<>();
            map.put("key", key);
            map.put("value", value);
            map.put("partition", record.partition());
            map.put("offset", record.offset());
            list.add(map);
        }
        return list;
    }
}

```

##### service.yml

We need to add the consumer startup and shutdown hooks and the KeyValueConsumer.

```
# Singleton service factory configuration/IoC injection
singletons:
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.kafka.ProducerStartupHook
  - com.networknt.kafka.ConsumerStartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.kafka.ProducerShutdownHook
  - com.networknt.kafka.ConsumerShutdownHook
- com.networknt.kafka.producer.LightProducer:
  - com.networknt.kafka.producer.TransactionalProducer
- com.networknt.kafka.consumer.LightConsumer:
  - com.networknt.kafka.KeyValueConsumer

```

##### kafka-consumer.yml

```
bootstrapServers: localhost:9092
isolationLevel: read_committed
enableAutoCommit: false
autoOffsetReset: earliest
autoCommitIntervalMs: 1000
keyDeserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
valueDeserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
# every consumer must have a groupId and it is retrieved from environment of server.yml
groupId: group1

```

##### ConsumerTopicGetHandler.java

This is the handler implementation. The topic is the path parameter and the response is a list of key/value pairs. 


```
package com.networknt.kafka.handler;

import com.networknt.config.JsonMapper;
import com.networknt.handler.LightHttpHandler;
import com.networknt.httpstring.ContentType;
import com.networknt.kafka.ConsumerStartupHook;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Headers;
import io.undertow.util.HttpString;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.http.HttpHeaders;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ConsumerTopicGetHandler implements LightHttpHandler {
    private static Logger logger = LoggerFactory.getLogger(ConsumerTopicGetHandler.class);

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        logger.debug("ConsumerTopicGetHandler start");
        String topic = exchange.getQueryParameters().get("topic").getFirst();
        logger.info("topic: " + topic);
        ConsumerStartupHook.consumer.subscribe(topic);
        List<Map<String, Object>> list = ConsumerStartupHook.consumer.poll();
        if(list != null) {
            exchange.getResponseHeaders().add(Headers.CONTENT_TYPE, "application/json");
            exchange.getResponseSender().send(JsonMapper.toJson(list));
        }
        exchange.endExchange();
    }
}

```

##### Test Consumer

After you have tested the producer, there should be some records in the Kafka test topic. Run the following command to retrieve the records from Kafka topic. 

```
curl -k 'https://localhost:8443/consumer/test'
```

### Summary

This example only touches the surface of the light-kafka capabilities. You can leverage many configuration combinations in the producer and consumer to build your handlers. However, the producer and consumer example is a good starting point if you are new to Kafka. 

