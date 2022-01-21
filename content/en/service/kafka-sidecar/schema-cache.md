---
title: "Schema Cache"
date: 2022-01-20T21:53:10-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When sending a request from the backend API to the Kafka Sidecar producer endpoint, most metadata in the [ProduceRequest][] object are optional, and only a list of [ProduceRecord][] is mandatory. Most users will only pass a list of records to the Kafka Sidecar to simplify the backend API, but we would recommend adding keySchemaId or keySchemaVersion for the Key and valueSchemaId or valueSchemaVersion for the Value to enable the schema cache on the sidecar to avoid a round trip to the schema-registry for each request. 

Here are the scenarios for the schema cache: 


### No cache

Suppose there is no shemaId and schemaVersion in the ProduceRequest,  the sidecar will search for the latest keySchema and valueSchema from the schema-registry based on the topic and a boolean value to indicate if the request is for key or value. As the latest schema for the key or value can be changed by any application given permission, we cannot cache it on the sidecar, and this requires the sidecar to do the query for each request, and in some cases, it will take up to 700 milliseconds to get the latest schema as the schema registry client will create a new connection for each request. 

### SchemaId

For most use cases, we have a topic with the same type of key and same type of value, and a key schema and a value schema are defined for the topic. You can find the keySchemaId and valueSchemaId from the control center or query the schema-registry. Once you know the keySchemaId and valueSchemaId, you can pass them in the ProduceRequest. The sidecar will use topicName + k + keySchemaId to cache the keySchema. And it will use topicName + v + valueSchemaId to cache the valueSchema. 

### SchemaVersion

It is hard for developers who don't have access to the control center and schema-registry to find the schemaId. Since the subject and schema version combination can uniquely identify a schema like a schemaId, we can cache with the subject and schemaVersion if schemaVersion is passed in the ProduceRequest.

There are two situations when caching with the schemaVersion: 

* If the subject is passed to in the ProduceRequest, the sidecar will use keySchemaSubject + keySchemaVersion to cache the keySchema and valueSchemaSubject + valueSchemaVersion to cache the valueSchema. 


* If the subject doesn't exist, then topicName + k will replace keySchemaSubject, and topicName + v will replace valueSchemaSubject for the cache key. 


### Performance

The performance difference is huge when using the schema cache on the sidecar. The throughput can increase from 50 times to 100 times in some of our tests with an enterprise Kafka cluster. 

### Example

Here is an example request to use the valueSchemaId to cache the valueSchema.

```
curl -k --location --request POST 'https://localhost:9443/producers/test1' \
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
    "valueSchemaId": 13
   
}'
```

Here is an example request to use the valueSchemaVersion to cache the valueSchema.

```
curl -k --location --request POST 'https://localhost:9443/producers/test1' \
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


[ProduceRequest]: https://github.com/networknt/light-kafka/blob/master/kafka-entity/src/main/java/com/networknt/kafka/entity/ProduceRequest.java
[ProduceRecord]: https://github.com/networknt/light-kafka/blob/master/kafka-entity/src/main/java/com/networknt/kafka/entity/ProduceRecord.java


