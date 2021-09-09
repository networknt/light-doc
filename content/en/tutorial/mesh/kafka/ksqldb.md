---
title: "Ksqldb Consumer Tutorial"
date: 2021-04-05T11:48:10-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

KsqlDB server is part of the Confluent Platform, and it provides the streams processor to hide the details of the Kafka streams. Users can create tables and streams on the KsqlDB server and subscribe to the KsqlDB server from the sidecar. Once the table or stream is changed, a rest call will be issued by the sidecar to the backend API to notify that new data received. 


### Kafka Topic

We will use the Confluent local services for this tutorial. After start the Confluent Platform locally. We need to create a topic called test with the following schemas for the key and value. 

key schema

```
{
  "$id": "http://example.com/myURI.schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "additionalProperties": false,
  "description": "Sample schema to help you get started.",
  "title": "key_test",
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
    "country": {
      "enum": [
        "CA",
        "US"
      ],
      "type": "string"
    },
    "firstName": {
      "description": "First Name",
      "type": "string"
    },
    "lastName": {
      "description": "Last Name",
      "type": "string"
    },
    "userId": {
      "description": "User Id",
      "type": "string"
    }
  },
  "title": "value_test",
  "type": "object"
}
```

The defined schemas will ensure that the data we produce to the test topic will be validated with the Kafka sidecar producer. 

### KsqlDB Stream

We are going to create a stream based on the Kafka topic test from the Control Center. 


```
CREATE STREAM TEST_STREAM (
    userId VARCHAR KEY, 
    firstName VARCHAR, 
    lastName VARCHAR, 
    country VARCHAR
) WITH (
  kafka_topic = 'test',
  value_format = 'JSON_SR'
);
```

Please note we are using "JSON_SR" here as value format. There are two JSON formats, JSON and JSON_SR. Both support serializing and deserializing JSON data. The latter offers integration with the Schema Registry, registering and retrieving JSON schemas while the former does not. These two formats are not byte compatible (you cannot read data produced by one by the other)


### Verify KsqlDB Stream

Now let's add some event message to the topic (test) we created above:

-  Start kafka-sidecar locally by referring to  [kafka-sidecar](/tutorial/kafka-sidecar/local-dev/)


-  produce some event message to "test" topic:

```text
curl --location --request POST 'https://localhost:8443/producers/test' \
--header 'Content-Type: application/json' \
--data-raw '{
    "records": [
        {
            "key": "1",
            "value": {
                "userId": "1111",
                 "firstName": "test1"
            }
        },
        {
            "key": "2",
            "value": {
                "userId": "2222",
                 "firstName": "test2"                
            }
        },
        {
            "key": "3",
            "value": {
                "userId": "3333",
                 "firstName": "test3"                       
            }
        }
    ]
}'
```

- From  the Kafka Control Center, in the navigation bar, click ksqlDB. In the Steams tab, we can see the "TEST_STREAM" we created above.


- Click the stream link (TEST_STREAM), and then click "Query stream" button.


- We can see the  query  running and result display as below:

![Stream result](/images/stream1.png)



