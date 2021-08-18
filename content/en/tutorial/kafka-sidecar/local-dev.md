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

To test with different schemas for validation and serialization/deserialization, we will create three topics. If you are using the Confluent Platform, it can be done from the control center interface. 

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

### Reactive Consumer


While producing some records to the test1 topic, the backend API will receive notifications from the sidecar. This can be found from the console log with the messages in the logging statements. 
