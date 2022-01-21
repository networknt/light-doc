---
title: "Traceability Id"
date: 2022-01-20T23:04:07-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

For each producer request to the sidecar, the sidecar will generate a correlationId and put it into each produce record header for the Kafka message. If users want to use open tracing, then a tracerId will be put into the header. These two values are used per request, and multiple messages share the same correlationId or tracerId. 

If an application wants to trace each message produced to Kafka, the traceabilityId can be used in each ProduceRecord. In most cases, the traceabilityId will be picked from the message key or value that can uniquely identify the message. 

Here is the source code of [ProduceRecord][] that contains an optional traceabilityId along with key and value. The backend API will populate the value for each record. If any record is missing the traceabilityId, the Kafka message header won't have this value. 


The following is a request with traceabilityId for two of the three records. 

```
curl --location --request POST 'https://localhost:9443/producers/test1' \
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
                "count": 1
            },
            "traceabilityId": "john-id"
        },
        {
            "key": "alex",
            "value": {
                "count": 2
            }
        }
    ],
    "valueSchemaVersion": 1
   
}'
```

In the produced Kafka messages, we can see that the alice and john will have the traceabilityId, and alex doesn't have a traceabilityId. 


Here are the headers for alice.

```
[
  {
    "key": "X-Correlation-Id",
    "stringValue": "rHhqbyUhTkqT7HXWk8qBHg"
  },
  {
    "key": "X-Traceability-Id",
    "stringValue": "alice-id"
  }
]
```

Here are the headers for john.

```
[
  {
    "key": "X-Correlation-Id",
    "stringValue": "vw061EzVR6CDB5pgISBL9w"
  },
  {
    "key": "X-Traceability-Id",
    "stringValue": "john-id"
  }
]
```

Here are the headers for alex.

```
[
  {
    "key": "X-Correlation-Id",
    "stringValue": "rHhqbyUhTkqT7HXWk8qBHg"
  }
]
```

You can see the message for alex doesn't have a traceabilityId because the request record for alex doesn't have one. 



[ProduceRecord]: https://github.com/networknt/light-kafka/blob/master/kafka-entity/src/main/java/com/networknt/kafka/entity/ProduceRecord.java


