---
title: "Sidecar Producer"
date: 2021-03-19T08:09:29-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

### Endpoint

```
  '/producers/{topic}':
    post:
      operationId: postData
      summary: Post a list of users to Kafka
      parameters:
        - name: topic
          in: path
          required: true
          description: The kafka topic name
          schema:
            type: string
      requestBody:
        description: message data
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProduceRequest'
      responses:
        '200':
          description: Data successfully produced.
        '400':
          description: Unexpected error
          content:
            application/json:
              schema:
                "$ref": "#/components/schemas/Status"
      security:
        - kafka_auth:
            - kafka:w


...


  schemas:
    ProduceRequest:
      type: object
      properties:
        keyFormat:
          type: integer
          enum: [
            0,
            1,
            2,
            3,
            4
          ]
        keySchema:
          type: string
        keySchemaId:
          type: integer
        keySchemaVersion:
          type: integer
        keySchemaSubject:
          type: string
        valueFormat:
          type: integer
          enum: [
            0,
            1,
            2,
            3,
            4
          ]
        valueSchema:
          type: string
        valueSchemaId:
          type: integer
        valueSchemaVersion:
          type: integer
        valueSchemaSubject:
          type: string
        records:
          type: array
          items:
            type: object
      required:
        - records


```

The above section is extracted from the openapi.yaml for the producer endpoint. It is a post request with the topic as a path parameter.  The request body contains the key schemaId or raw schema and value schemaId or raw schema as optional fields. If schema info is missing from the request body, the producer will use the latest version of the schema defined for the topic key and value. A list of records is mandatory for the producer. 

### Configuration

The kafka-producer.yml is the configuration file for the producer in Kafka sidecar. The following is an example. 

```
---
# Generic configuration for Kafka producer.
properties:
  key.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
  value.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
  acks: ${kafka-producer.acks:all}
  bootstrap.servers: localhost:9092
  buffer.memory: 33554432
  retries: ${kafka-producer.retries:3}
  batch.size: 16384
  linger.ms: 1
  max.in.flight.requests.per.connection: ${kafka-producer.max.in.flight.requests.per.connection:5}
  enable.idempotence: ${kafka-producer.enable.idempotence:false}
  # security configuration for enterprises
  # security.protocol: SASL_SSL
  # sasl.mechanism: PLAIN
  # sasl.jaas.config: "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"username\" password=\"password\";"
  # ssl.endpoint.identification.algorithm: ""
  # client.rack:

# The default topic for the producer. Only certain producer implementation will use it.
topic: portal-event
# if open tracing is enable. traceability, correlation and metrics should not be in the chain if opentracing is used.
injectOpenTracing: ${client.injectOpenTracing:false}
# inject serviceId as callerId into the http header for metrics to collect the caller. The serviceId is from server.yml
injectCallerId: ${client.injectCallerId:false}
# schema registry url
schemaRegistryUrl: http://localhost:8081
# schema registry identity cache size
schemaRegistryCache: 100

```

There are several errors that might happen with the wrong configuration of the kafka-producer.yml. 

* Cluster authorization failed

This might be due to the idempotent enabled and the permission is not enabled. Set the enable.idempotence: false will resolve the problem. 

* Messages are rejected since there are fewer in-sync replicas than required

This might due to the acks: all in the config file with the topic only has one partition. Change the configuration to acks: '1' will resolve the problem.


We also need to update the service.yml to enable producer startup and shutdown hooks. 

```
- com.networknt.server.StartupHookProvider:
  - com.networknt.mesh.kafka.ProducerStartupHook
  # - com.networknt.mesh.kafka.ConsumerStartupHook
  # - com.networknt.mesh.kafka.KsqldbConsumerStartupHook
  - com.networknt.mesh.kafka.ReactiveConsumerStartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.mesh.kafka.ProducerShutdownHook
  # - com.networknt.mesh.kafka.ConsumerShutdownHook
  # - com.networknt.mesh.kafka.KsqldbConsumerShutdownHook
  - com.networknt.mesh.kafka.ReactiveConsumerShutdownHook
- com.networknt.kafka.producer.NativeLightProducer:
  - com.networknt.kafka.producer.SidecarProducer

```

### Certificate

When SSL/TLS is enabled for the Kafka servers, we need to get the certificate and import to the cacerts file under your jdk folder. 

```
cd jdk-home/lib/security
keytool -import alias kafka -keystore cacerts -file yourcert.cer
```

### Format

We can produce the message to a Kafka topic with five different formats for schema validation and serialization. 

From the schema above, you can see that the keyFormat and valueFormat are integers. There is a Java Enum that you can use in the programming if your API is in Java. Otherwise, here is the mapping. 

0 -> binary
1 -> json
2 -> avro
3 -> jsonschema
4 -> protobuf


##### binary



##### jsonschema

The key and value will use JSON schema to validate and serialize the producer records.

Before using this producer format, we need to create a topic test1 and define the JSON schema for the key and value. If you are using the Confluent Platform, it can be done from the control center interface. 

Here is the schema for the key and the schema ID is 1. 

```
{
  "$id": "http://example.com/myURI.schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "additionalProperties": false,
  "description": "Sample schema to help you get started.",
  "title": "key_test1",
  "type": "string"
}
```

Here is the schema for the value and the schema ID is 2.

```
{
  "$id": "http://example.com/myURI.schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "additionalProperties": false,
  "description": "Sample schema to help you get started.",
  "properties": {
    "count": {
      "description": "The integer type is used for integral numbers.",
      "type": "integer"
    }
  },
  "title": "value_test1",
  "type": "object"
}
```

To produce some messages to the test1 topic, we can issue a curl command. 

```
curl -k --location --request POST 'https://localhost:8443/producers/test1' \
--header 'Content-Type: application/json' \
--data-raw '{"key_schema_id":1,"value_schema_id":2,"records":[{"key":"alice","value":{"count":2}},{"key":"john","value":{"count":1}},{"key":"alex","value":{"count":2}}]}'
```

Result: 

```
{"offsets":[{"partition":0,"offset":495894},{"partition":0,"offset":495895},{"partition":0,"offset":495896}],"key_schema_id":1,"value_schema_id":2,"requestStatus":"OK"}
```

##### avro

The key and value will use Avro to validate and serialize the producer records.

Before using this producer format, we need to create a topic test2 and define the Avro schema for the key and value. If you are using the Confluent Platform, it can be done from the control center interface. 

Here is the schema for the key and the schema ID is 3. 

```
"string"
```

Here is the schema for the value and the schema ID is 5.

```
{
  "doc": "Sample schema to help you get started.",
  "fields": [
    {
      "doc": "The int type is a 32-bit signed integer.",
      "name": "count",
      "type": "int"
    }
  ],
  "name": "value",
  "namespace": "com.mycorp.mynamespace",
  "type": "record"
}
```

To produce some messages to the test2 topic, we can issue a curl command. 

```
curl -k --location --request POST 'https://localhost:8443/producers/test2' \
--header 'Content-Type: application/json' \
--data-raw '{"key_schema_id":3,"value_schema_id":5,"records":[{"key":"Alex","value":{"count":0}},{"key":"Alice","value":{"count":1}},{"key":"Bob","value":{"count":2}}]}'
```

Result:

```
{"offsets":[{"partition":0,"offset":15},{"partition":0,"offset":16},{"partition":0,"offset":17}],"key_schema_id":3,"value_schema_id":5,"requestStatus":"OK"}
```


##### protobuf

The key and value will use protobuf to validate and serialize the producer records.

Before using this producer format, we need to create a topic test3 and define the Protobuf schema for the key and value. If you are using the Confluent Platform, it can be done from the control center interface. 

Here is the schema for the key and the schema ID is 6. 

```
syntax = "proto3";
package com.mycorp.mynamespace;

message key_test3 {
  string name = 1;
}

```

Here is the schema for the value and the schema ID is 7.

```
syntax = "proto3";
package com.mycorp.mynamespace;

message value_test3 {
  int32 count = 1;
  string name = 2;
}

```

To produce some messages to the test3 topic, we can issue a curl command.

```
curl -k --location --request POST 'https://localhost:8443/producers/test3' \
--header 'Content-Type: application/json' \
--data-raw '{"key_schema_id":6,"value_schema_id":7,"records":[{"key":{"name": "Alex"},"value":{"count":0}},{"key":{"name": "Alice"},"value":{"count":1}},{"key":{"name":"Bob"},"value":{"count":2}}]}'
```

Result:

```
{"offsets":[{"partition":0,"offset":7},{"partition":0,"offset":8},{"partition":0,"offset":9}],"key_schema_id":6,"value_schema_id":7,"requestStatus":"OK"}
```

### Latest Schema

Suppose you want to produce the Kafka topic using the latest version of schema defined for the topic key and value; you can provide the records like the following without any additional metadata. 

To produce messages to the test1 topic, use the following curl command.

```
curl -k --location --request POST 'https://localhost:8443/producers/test1' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":"alice","value":{"count":2}},{"key":"john","value":{"count":1}},{"key":"alex","value":{"count":2}}]}'
```

### Schema Subject

The above commands assume that we have the schemas defined for the key and value for the topic. It means we can only produce one type of data to the topic with only one schema to validate and serialize it. If we want to produce different data types for the same topic, we cannot do that with the above configuration. 

For example, we have multiple event types for event sourcing and want to produce all event types to the same event topic. In that case, we need to define the schema based on the subject. 

For example, we have UserCreatedEvent, UserUpdatedEvent and UserDeletedEvent with corresponding schemas defined with subject name UserCreatedEvent, UserUpdatedEvent and UserDeletedEvent.

To produce messages to the test1 topic, use the following curl command.

```
curl -k --location --request POST 'https://localhost:8443/producers/event' \
--header 'Content-Type: application/json' \
--data-raw '{"subject": "UserCreatedEvent", "records":[{"key":"alice","value":{"count":2}},{"key":"john","value":{"count":1}},{"key":"alex","value":{"count":2}}]}'
```

