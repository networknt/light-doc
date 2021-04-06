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
        keySchema:
          type: string
        keySchemaId:
          type: string
        valueSchema:
          type: string
        valueSchemaId:
          type: string
        records:
          type: array
          items:
            type: object
      required:
        - records


```

The above section is extracted from the openapi.yaml for the producer endpoint. It is a post request with the topic as a path parameter.  The request body contains the key schemaId or raw schema and value schemaId or raw schema as optional fields. If schema info is missing from the request body, the producer will use the latest version of the schema defined for the topic key and value. A list of records is mandatory for the producer. 


### Format

We can produce the message to a Kafka topic with three different formats for schema validation and serialization. 

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

