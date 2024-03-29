---
title: "Local Development with Default Configuration"
date: 2021-08-16T08:09:36-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This tutorial will run the sidecar and backend API with local default configuration for producer and reactive consumer. 

### Confluent Local

As we plan to run everything locally, we need to install and start the Confluent local. I am using confluent 6.1 on Ubuntu Linux 20.10 desktop. 

```
confluent local services start
```

### Create Test Topics

To test with different schemas for validation and serialization/deserialization, we will create three topics (test1, test2, test3). If you are using the Confluent Platform, it can be done from the control center interface.

To verify kafka sidecar Reactive Consumer DLQ feature,   we can create a Dead Letter Queue(DLQ): test1.dlq


##### test1

For test1 topic, json schema will be used to define the key and value as the following (Type: JSON Schema).

key schema

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

value schema

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

To send a request with a wrong record in the list. 

```
{
    "records": [
        {
            "key": "alice",
            "value": {
                "count": 2
            },
            "traceabilityId": "alice-id"
        },
        {
            "key": "john",
            "value": {
                "count": "1"
            },
            "traceabilityId": "john-id"
        },
        {
            "value": {
                "count": 2
            }
        }
    ],
    "valueSchemaVersion": 1
   
}
```

Clearly, the count for John is not an integer but a string. The response from the server is 

```
{
    "statusCode": 400,
    "code": "ERR12206",
    "message": "SERIALIZE_SCHEMA_EXCEPTION",
    "description": "Unexpected exception in serializing jsonschema format with message #/count: expected type: Integer, found: String",
    "severity": "ERROR"
}
```


##### test2


For test2 topic, avro will be used to define the key and value as the following (Type: Avro).

key schema

```
"string"
```

value schema

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

##### test3


For test3 topic, protobuf will be used to define the key and value as the following (Type: Protobuf).

key schema

```
syntax = "proto3";
package com.mycorp.mynamespace;

message key_test3 {
  string name = 1;
}

```

value schema

```
syntax = "proto3";
package com.mycorp.mynamespace;

message value_test3 {
  int32 count = 1;
  string name = 2;
}

```

##### test4

for test4 topic, we have different formats for the key and value. The value will be using jsonschema and key the will have no schema with string format only. 

In most of the cases, we use schemas for both key and value so that the kafka-sidecar can decide how to validate and serialize the record based on the schema definition (jsonschema, avro or protobuf). However, there are cases that key will be just plain string especially when the consumer is expecting the key is a string intead of an object with a schema. 

The following is the schema for the value and it is the same as test1 value.

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



### Sidecar

The sidecar supports different configurations based on the business logic of the backend API. We will use the default configuration for this first tutorial to start it in an IDE with debug mode.

Checkout the light-mesh repo from Github networknt org.


```
cd ~/networknt

git@github.com:networknt/kafka-sidecar.git
```

Start the kafka-sidecar with IntelliJ Idea. 


### Backend

By default, the sidecar will enable the reactive consumer during the startup based on the default configuration. Once a message is received from Kafka on topic test1, the sidecar will be notified, and it will invoke the backend API `/kafka/records` endpoint. An example of backend API can be found in the light-example-4j repo. 

```
cd ~/networknt

git clone git@github.com:networknt/light-example-4j.git
```

The sidecar-backend is located in https://github.com/networknt/light-example-4j/tree/release/kafka/sidecar-backend


Start the sidecar-backend with IntelliJ Idea.


### Producer

To produce some messages to the test1 topic, we can issue a curl command. 

```
curl -k --location --request POST 'https://localhost:8443/producers/test1' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":"alice","value":{"count":2}},{"key":"john","value":{"count":1}},{"key":"alex","value":{"count":2}}]}'
```

Result: 

```
{"offsets":[{"partition":0,"offset":495894},{"partition":0,"offset":495895},{"partition":0,"offset":495896}],"key_schema_id":1,"value_schema_id":2,"requestStatus":"OK"}
```

To produce some messages to the test2 topic, we can issue a curl command. 

```
curl -k --location --request POST 'https://localhost:8443/producers/test2' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":"Alex","value":{"count":0}},{"key":"Alice","value":{"count":1}},{"key":"Bob","value":{"count":2}}]}'
```

Result:

```
{"offsets":[{"partition":0,"offset":15},{"partition":0,"offset":16},{"partition":0,"offset":17}],"key_schema_id":3,"value_schema_id":5,"requestStatus":"OK"}
```

To produce some messages to the test3 topic, we can issue a curl command.

```
curl -k --location --request POST 'https://localhost:8443/producers/test3' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":{"name": "Alex"},"value":{"count":0}},{"key":{"name": "Alice"},"value":{"count":1}},{"key":{"name":"Bob"},"value":{"count":2}}]}'
```

Result:

```
{"offsets":[{"partition":0,"offset":7},{"partition":0,"offset":8},{"partition":0,"offset":9}],"key_schema_id":6,"value_schema_id":7,"requestStatus":"OK"}
```

To produce some messages to the test4 topic, we can issue a curl command.

```
curl -k --location --request POST 'https://localhost:8443/producers/test4' \
--header 'Content-Type: application/json' \
--data-raw '{"records":[{"key":"alice","value":{"count":2}},{"key":"john","value":{"count":1}},{"key":"alex","value":{"count":2}}]}'

```

Result:

```
{"offsets":[{"partition":0,"offset":6},{"partition":0,"offset":7},{"partition":0,"offset":8}],"value_schema_id":24,"requestStatus":"OK"}
```

To produce some messages with one invaid entry against the schema like the following message body with the second entry count is "abc" instead of integer. 

```
curl -k --location --request POST 'http://localhost:9443/producers/test1' \
--header 'Content-Type: application/json' \
--header 'X-Traceability-Id: common-id' \
--data-raw '{
    "records": [
        {
            "key": "alice",
            "value": {
                "count": 2
            },
            "traceabilityId": "alice-id"
        },
        {
            "key": "john",
            "value": {
                "count": "abc"
            },
            "traceabilityId": "john-id"
        },
        {
            "value": {
                "count": 2
            }
        }
    ]
}'
```

We go the following error message from the sidecar. It means we have to resend the entire back again after fixing the error entry in the batch. 

```
{
    "statusCode": 400,
    "code": "ERR12206",
    "message": "SERIALIZE_SCHEMA_EXCEPTION",
    "description": "Unexpected exception in serializing jsonschema format with message #/count: expected type: Integer, found: String",
    "severity": "ERROR"
}
```


### Reactive Consumer


While producing some records to the test1 topic, the backend API will receive notifications from the sidecar. This can be found from the console log with the messages in the logging statements. 

And from kafka sidecar console, we can see the logs:

```text
12:42:20.801 [XNIO-1 task-1]  5qgC8sVTRamrAjRHueNdyw INFO  c.s.e.e.m.k.h.ProducersTopicPostHandler:154 handleRequest - ProducerTopicPostHandler handleRequest start with topic test1
12:42:21.206 [pool-3-thread-1]   INFO  c.s.e.e.m.k.ReactiveConsumerStartupHook$1:172 onCompletion - Send a batch to the backend API
12:42:21.222 [pool-3-thread-1]   INFO  c.s.e.e.m.k.ReactiveConsumerStartupHook$1:186 onCompletion - Got successful response from the backend API
```

In the sample backend API, the first event for each message, it will be mark as process failed; And the  kafka sidecar Reactive Consumer will send the message to kafka DLQ (test1.dlq).

```text
 RecordProcessedResult rpr = new RecordProcessedResult(record, false, sw.toString());
```

