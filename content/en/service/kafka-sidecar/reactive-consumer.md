---
title: "Reactive Consumer"
date: 2021-03-26T10:59:04-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The reactive consumer is similar to the Kafka Streams. The backend API/APP exposes an API endpoint to receive data once the sidecar detects new messages in the Kafka topic(s). Once the backend consumes the messages, a list of RecordProcessedResult returns to the sidecar to write dead letter topic if any and sidecar audit topic. 

### Configuration

The config file kafka-consumer.yml is shared by both the active consumer and the reactive consumer. There are four sections in the file: 

* Kafka Connectivity

```
properties:
  bootstrap.servers: ${kafka-consumer.bootstrap.servers:localhost:9092}
  # Consumer will use the schema for deserialization from byte array
  key.deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
  value.deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
  # As the control pane or API to access admin endpoint for commit, this value should be false.
  enable.auto.commit: false
  auto.offset.reset: ${kafka-consumer.auto.offset.reset:earliest}
  group.id: ${kafka-consumer.group.id:group1}
  schema.registry.url: ${kafka-consumer.schema.registry.url:http://localhost:8081}
  # security configuration for enterprise deployment
  # security.protocol: ${kafka-consumer.security.protocol:SASL_SSL}
  # sasl.mechanism: ${kafka-consumer.sasl.mechanism:PLAIN}
  # sasl.jaas.config: "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"${kafka-consumer.username:username}\" password=\"${kafka-consumer.password:password}\";"
  # ssl.endpoint.identification.algorithm: ${kafka-consumer.ssl.endpoint.identification.algorithm:algo-name}
  # client.rack: ${kafka-consumer.client.rack:rack-name}
  # basic authentication user:pass for the schema registry
  # basic.auth.user.info: ${kafka-consumer.username:username}:${kafka-consumer.password:password}
  # basic.auth.credentials.source: ${kafka-consumer.basic.auth.credentials.source:USER_INFO}
```

This section contains all the properties used to connect to the Kafka Cluster. It is an open section, and users can add more properties based on the Kafka cluster setup. If additional properties are added to this section and need to be externalized, please name it with kafka-consumer.propertyName in the template. Please note that the username and password are used in Kafka connection SASL and schema registry basic authentication. 


* Consumer Common

```
# common configuration properties between active and reactive consumers
# Indicator if the dead letter topic is enabled.
deadLetterEnabled: ${kafka-consumer.deadLetterEnabled:true}
# The extension of the dead letter queue(topic) that is added to the original topic to form the dead letter topic
deadLetterTopicExt: ${kafka-consumer.deadLetterTopicExt:.dlq}
# Indicator if the audit is enabled.
auditEnabled: ${kafka-consumer.auditEnabled:true}
# The consumer audit topic name
auditTopic: ${kafka-consumer.auditTopic:sidecar-audit}
# Indicate if the NoWrapping Avro converter is used. This should be used for avro schema with data type in JSON.
useNoWrappingAvro: ${kafka-consumer.useNoWrappingAvro:false}

```

This section is used for both consumers to define the dead letter and audit log. Also it gives users an option to replace the Avro to JSON converter to unwrap the type in JSON. 


* Reactive Consumer

```
# Reactive Consumer Specific Configuration
# The topic that is going to be consumed. For reactive consumer only in the kafka-sidecar.
# If two or more topics are going to be subscribed, concat them with comma without space.
# topic: sidecar-test
topic: ${kafka-consumer.topic:test1}
# the format of the key optional
keyFormat: ${kafka-consumer.keyFormat:jsonschema}
# the format of the value optional
valueFormat: ${kafka-consumer.valueFormat:jsonschema}
# Waiting period in millisecond to poll another batch
waitPeriod: ${kafka-consumer.waitPeriod:10000}
# Backend API host
backendApiHost: ${kafka-consumer.backendApiHost:https://localhost:8444}
# Backend API path
backendApiPath: ${kafka-consumer.backendApiPath:/kafka/records}
```

Defines the topic, default format, wait and backend API endpoint for the Reactive Consumer.

* Active Consumer

```
# Active Consumer Specific Configuration and the reactive consumer also depends on these properties
# default max consumer threads to 50.
maxConsumerThreads: ${kafka-consumer.maxConsumerThreads:50}
# a unique id for the server instance, if running in a Kubernetes cluster, use the container id environment variable
serverId: ${kafka-consumer.serverId:id}
# maximum number of bytes message keys and values returned. Default to 64*1024*1024
requestMaxBytes: ${kafka-consumer.requestMaxBytes:67108864}
# The maximum total time to wait for messages for a request if the maximum number of messages hs not yet been reached.
requestTimeoutMs: ${kafka-consumer.requestTimeoutMs:1000}
# Minimum bytes of records to accumulate before returning a response to a consumer request. Default 10MB
fetchMinBytes: ${kafka-consumer.fetchMinBytes:10000000}
# amount of idle time before a consumer instance is automatically destroyed.
instanceTimeoutMs: ${kafka-consumer.instanceTimeoutMs:300000}
# Amount of time to backoff when an iterator runs out of date.
iteratorBackoffMs: ${kafka-consumer.iteratorBackoffMs:50}

```

Active consumer configuration shared with the Reactive consumer as the Reacive Consumer is based on the Active Consumer. 


### Backend API Specification

As the sidecar will invoke an API of the backend once the data is available, the backend API must have an endpoint defined with the following OpenAPI 3.0 specification. The latest version can be found [here](https://github.com/networknt/model-config/blob/master/rest/kafka-sidecar-backend/openapi.yaml).

```
---
openapi: "3.0.0"
info:
  version: "1.0.0"
  title: "Sidecar Backend"
  license:
    name: "MIT"
servers:
- url: "http://backend.sidecar.networknt.com"
paths:
  /kafka/records:
    post:
      summary: "Post a list of Kafka records"
      operationId: "postRecords"
      requestBody:
        description: "An array of consumer records"
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Records"
      security:
      - sidecar_auth:
        - "write:sidecar"
      responses:
        "200":
          description: "A list of RecordProcessedResult"
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/RecordProcessedResult'
        "400":
          description: "Error processing the record. Need sidecar to rollback and retry after waiting period"
          
components:
  securitySchemes:
    sidecar_auth:
      type: "oauth2"
      description: "This API uses OAuth 2 with the client credential grant flow."
      flows:
        clientCredentials:
          tokenUrl: "https://localhost:6882/token"
          scopes:
            write:sidecar: "Add records"
  schemas:
    Records:
      type: "array"
      items:
        $ref: "#/components/schemas/Record"
    Record:
      type: "object"
      properties:
        topic:
          type: "string"
        key:
          oneOf:
            - $ref: "#/components/schemas/StringKey"
            - $ref: "#/components/schemas/ObjectKey"
        value:
          oneOf:
            - $ref: "#/components/schemas/StringValue"
            - $ref: "#/components/schemas/ObjectValue"
        header:
          type: "object"
        partition:
          type: "integer"
          minimum: 0
        offset:
          type: "number"
          minimum: 0
    StringKey:
      type: "string"
    ObjectKey:
      type: "object"
    StringValue:
      type: "string"
    ObjectValue:
      type: "object"
    ConsumerRecord:
      type: "object"
      properties:
        topic: 
          type: "string"
        key: 
          oneOf:
            - $ref: "#/components/schemas/StringKey"
            - $ref: "#/components/schemas/ObjectKey"
        value:
          oneOf:
            - $ref: "#/components/schemas/StringValue"
            - $ref: "#/components/schemas/ObjectValue"
        partition: 
          type: "integer"
          minimum: 0
        offset: 
          type: "number"
          minimum: 0
    RecordProcessedResult:
      type: "object"
      properties:
        consumerRecord:
          $ref: "#/components/schemas/ConsumerRecord"
        processed: 
          type: "boolean"
        stacktrace:
          type: "string"
      required: 
        - consumerRecord
        - processed
```

The endpoint path is configurable on the kafka-sidecar, so you can customize the endpoint for your backend API implementation. Make sure you return a list of RecordProcessedResult objects to the sidecar to indicate the process status for each record. 


### Backend API Implementation


The following is a simple light-4j handler implementation to output he received messages to the console via logging statements. 

```
package com.networknt.mesh.kafka.backend.handler;

import com.networknt.body.BodyHandler;
import com.networknt.config.Config;
import com.networknt.config.JsonMapper;
import com.networknt.handler.LightHttpHandler;
import com.networknt.kafka.entity.ConsumerRecord;
import com.networknt.kafka.entity.RecordProcessedResult;
import io.undertow.server.HttpServerExchange;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
For more information on how to write business handlers, please check the link below.
https://doc.networknt.com/development/business-handler/rest/
*/
public class KafkaRecordsPostHandler implements LightHttpHandler {

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<RecordProcessedResult> results = new ArrayList<>();
        List<Map<String, Object>> records = (List<Map<String, Object>>)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        for(int i = 0; i < records.size(); i++) {
            ConsumerRecord record = Config.getInstance().getMapper().convertValue(records.get(i), ConsumerRecord.class);
            logger.debug("topic = " + record.getTopic());
            logger.debug("partition = " + record.getPartition());
            logger.debug("offset = " + record.getOffset());
            logger.debug("key = " + record.getKey());
            logger.debug("value = " + record.getValue());
            Map<String, String> headerMap = record.getHeaders();
            if(headerMap != null && headerMap.size() > 0) {
                headerMap.entrySet().forEach(entry -> {
                    logger.debug("header key = " + entry.getKey() + " header value = " + entry.getValue());
                });
            }

            if(i == 1) {
                // simulate runtime exception that causes record cannot be processed successfully and ask
                // sidecar to put the record to the dead letter queue.
                try {
                    int j = 10 / 0;
                } catch (RuntimeException e) {
                    StringWriter sw = new StringWriter();
                    PrintWriter pw = new PrintWriter(sw);
                    e.printStackTrace(pw);
                    RecordProcessedResult rpr = new RecordProcessedResult(record, false, sw.toString());
                    results.add(rpr);
                }
            } else {
                // simulate processed successfully.
                RecordProcessedResult rpr = new RecordProcessedResult(record, true, null);
                results.add(rpr);
            }
        }
        exchange.setStatusCode(200);
        exchange.getResponseSender().send(JsonMapper.toJson(results));
    }
}

```

As you can see, we generate the RuntimeException for the second record to simulate the dead letter process on the sidecar. The example application can be found on the Github.com in /networknt/light-example-4j/kafka/sidecar-backend/

### Test Within IDE


### Test with Docker


### Test in Kubernetes




