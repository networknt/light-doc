---
title: "Reactive Consumer Producer"
date: 2022-02-18T16:10:29-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This tutorial gives users a reference implementation of the reactive consumer service. It receives a batch, transforms the value and then produces the new value to another topic on the same Kafka cluster. We should do the transformation with Kafka streams as it is designed for this kind of task. In real business scenarios, the consumer might populate the transformed result to a database or invoke another API. 


### Confluent Platform

Start Confluent platform with by following the [Confluent Docker][] if you have docker installed. Otherwise, you can start the [Confluent Platform] locally on a Linux machine. 



[Confluent Docker]: /tutorial/kafka-sidecar/confluent-docker/
[Confluent Platform]: /tutorial/kafka-sidecar/confluent-local/


### Topics

We need to create two topics to demo the consumer and producer in the same backend application.

The first topic is called the original.account, and the second is called the transformed.account.

You can create both topics with the control center web interface. The control center can be accessed with http://localhost:9021


For original.account topic, create a value JSON schema like the following. 


Value schema for the original.account topic

```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/myURI.schema.json",
  "title": "SampleRecord",
  "description": "Sample schema to help you get started.",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "accountNo": {
      "type": "string",
      "description": "The account number."
    },
    "firstName": {
      "type": "string",
      "description": "The first name of the account holder."
    },
    "lastName": {
      "type": "string",
      "description": "The last name of the account holder."
    }
  }
}
```

For the transformed.account topic, create a value JSON schema like the following.

Value schema for the transformed.account topic

```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/myURI.schema.json",
  "title": "SampleRecord",
  "description": "Sample schema to help you get started.",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "accountNo": {
      "type": "string",
      "description": "The account number."
    },
    "firstName": {
      "type": "string",
      "description": "The first name of the account holder."
    },
    "lastName": {
      "type": "string",
      "description": "The last name of the account holder."
    },
    "fullName": {
      "type": "string",
      "description": "The account holder full name."
    }
  }
}

```

### Start the backend in IDE

For developers who want to understand the interaction between the Kafka-sidecar and the backend API, you can run both servers in IDE with debug mode enabled. Later, we will start both servers with a docker-compose to demo the sidecar pattern. 

The backend API is located in GitHub at https://github.com/networknt/light-example-4j/tree/master/kafka-sidecar/reactive-consumer-producer

Check it out and start it within the IDE with the default config. It will bind to port 8443 with HTTPS. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j
git checkout master
cd kafka-sidecar/reactive-consumer-producer
```

### Start the sidecar in IDE

The sidecar is located in GitHub at https://github.com/networknt/kafka-sidecar

Check it out to the local workspace.


```
cd ~/networknt
git clone https://github.com/networknt/kafka-sidecar.git
cd kafka-sidecar
```

Start it within the IDE with the following Java option. 

```
-Dlight-4j-config-dir=config/reactive-consumer-producer
```

Start the sidecar in the IDE with debug mode and it will bind to port 9443.

As the original.account topic is newly created and it is empty, there shouldn't any activity between the sidecar and the backend. 

### Produce to original.account

To trigger the processing flow, we need to produce some records into the original.account topic. Also, we need to carefully design the records to control how the transformation and reproduction are done. 

Here is the body of the request in an easy-to-read format. 

```
{
    "records": [
        {
            "key": "1",
            "value": {
                "accountNo": "1",
                "firstName": "Alex",
                "lastName": "King"
            },
            "traceabilityId": "1"
        },
        {
            "key": "2",
            "value": {
                "accountNo": "2",
                "firstName": "John",
                "lastName": "Doe"
            }
        },
        {
            "value": {
                "accountNo": "3",
                "firstName": "Bob",
                "lastName": "Cat"
            }
        }
    ],
    "valueSchemaVersion": 1
}
```

Notice that the third record doesn't have a key, as the key is optional. We don't produce the record without a key in the backend handler to the transformed.account topic. So only the first two records will be shown up in the transformed.account topic.

You can see that the first record has a traceabilityId and the rest don't have this field. The backend handler will pass the traceabilityId to the audit entry and newly produced record. The records that don't have the traceabilityId will pick the accountNo from the value to add the traceabilityId. 

The backend will always receive a correlationId for each record in the batch as the side consumer will add it. There will be a logging statement to associate the correlationId and the traceabilityId in logs.

Here is a curl command to post the above body to the sidecar.


```
curl --location --request POST 'http://localhost:9443/producers/original.account' \
--header 'Content-Type: application/json' \
--data-raw '{
    "records": [
        {
            "key": "1",
            "value": {
                "accountNo": "1",
                "firstName": "Alex",
                "lastName": "King"
            },
            "traceabilityId": "1"
        },
        {
            "key": "2",
            "value": {
                "accountNo": "2",
                "firstName": "John",
                "lastName": "Doe"
            }
        },
        {
            "value": {
                "accountNo": "3",
                "firstName": "Bob",
                "lastName": "Cat"
            }
        }
    ],
    "valueSchemaVersion": 1
}'
```

### Logs

If you start the kafka-sidecar before the reactive-consumer-producer backend, you will repeatedly see the following logging statement until the backend is up and running.

```
2022-02-20T13:44:48.690 [pool-2-thread-1]   INFO  c.n.m.k.ReactiveConsumerStartupHook:190 readRecords - Could not borrow backend connection , will retry !!!
2022-02-20T13:44:49.691 [pool-2-thread-1]   INFO  c.n.m.k.ReactiveConsumerStartupHook:190 readRecords - Could not borrow backend connection , will retry !!!
```

Once the backend is started, both IDEs will have no logs moving as both are waiting for messages from the original.account topic. 

Run the curl command above or send the request from the Postman, the log in the sidecar is shown below. 

```
2022-02-20T13:53:55.923 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/original.account, base path is set to: null
2022-02-20T13:53:55.923 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/original.account
2022-02-20T13:53:55.924 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
2022-02-20T13:53:55.924 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
2022-02-20T13:53:55.937 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.config.Config$FileConfigImpl:649 loadJsonMapConfigWithSpecificConfigLoader - Trying to load mask with extension yaml, yml or json by using default loading method.
2022-02-20T13:53:55.937 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.config.Config$FileConfigImpl:508 getConfigStream - Unable to load config from externalized folder for mask.yml in config/reactive-consumer-producer
2022-02-20T13:53:55.938 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.config.Config$FileConfigImpl:521 getConfigStream - Trying to load config from classpath directory for file mask.yml
2022-02-20T13:53:55.938 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.config.Config$FileConfigImpl:533 getConfigStream - Config loaded from default folder for mask.yml
2022-02-20T13:53:55.940 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/original.account, base path is set to: null
2022-02-20T13:53:55.940 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/original.account
2022-02-20T13:53:55.961 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw WARN  com.networknt.schema.JsonMetaSchema:387 newValidator - Unknown keyword components - you should define your own Meta Schema. If the keyword is irrelevant for validation, just use a NonValidationKeyword
2022-02-20T13:53:55.967 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "original.account", "original.account", topic)
2022-02-20T13:53:55.969 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.970 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.970 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.971 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:55.971 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[0])
2022-02-20T13:53:55.971 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[1])
2022-02-20T13:53:55.971 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[2])
2022-02-20T13:53:55.971 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.972 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.973 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.973 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.973 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.973 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:55.974 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic original.account
2022-02-20T13:53:56.073 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 85
2022-02-20T13:53:56.074 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 1 with correlation Id n1eYKlNmQNGcBwVcefeW4g
2022-02-20T13:53:56.087 [XNIO-1 task-1]  BeqC9QnGT_C_oTAroR06fw DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 14
2022-02-20T13:53:56.097 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 4
2022-02-20T13:53:56.098 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 125

2022-02-20T13:53:56.188 [pool-3-thread-4]   DEBUG c.n.m.k.ReactiveConsumerStartupHook$1:125 onCompletion - polled records size = 3
2022-02-20T13:53:56.190 [pool-3-thread-4]   INFO  c.n.m.k.ReactiveConsumerStartupHook$1:132 onCompletion - Send a batch to the backend API
2022-02-20T13:53:56.313 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
2022-02-20T13:53:56.314 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
2022-02-20T13:53:56.314 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
2022-02-20T13:53:56.314 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
2022-02-20T13:53:56.315 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
2022-02-20T13:53:56.315 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
2022-02-20T13:53:56.316 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "transformed.account", "transformed.account", topic)
2022-02-20T13:53:56.316 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.316 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.317 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.317 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:56.317 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records[0])
2022-02-20T13:53:56.317 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:56.317 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.318 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
2022-02-20T13:53:56.318 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.keySchemaVersion)
2022-02-20T13:53:56.318 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.318 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.318 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.319 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.319 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.319 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.319 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.319 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic transformed.account
2022-02-20T13:53:56.325 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 6
2022-02-20T13:53:56.325 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 1 with correlation Id n1eYKlNmQNGcBwVcefeW4g
2022-02-20T13:53:56.326 [XNIO-1 task-1]  lT8vv7mySeSyvOmSQBTwdA DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 1
2022-02-20T13:53:56.329 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 1
2022-02-20T13:53:56.329 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 10
2022-02-20T13:53:56.345 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
2022-02-20T13:53:56.345 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
2022-02-20T13:53:56.345 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
2022-02-20T13:53:56.345 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
2022-02-20T13:53:56.346 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
2022-02-20T13:53:56.346 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
2022-02-20T13:53:56.346 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "transformed.account", "transformed.account", topic)
2022-02-20T13:53:56.347 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.347 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.348 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.348 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}], {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:56.348 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records[0])
2022-02-20T13:53:56.348 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}], {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
2022-02-20T13:53:56.348 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.349 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
2022-02-20T13:53:56.349 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.keySchemaVersion)
2022-02-20T13:53:56.349 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.349 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.349 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.350 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.350 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.350 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.350 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
2022-02-20T13:53:56.350 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic transformed.account
2022-02-20T13:53:56.351 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 0
2022-02-20T13:53:56.351 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 2 with correlation Id 7wCz993MR1aAzy3sg-jkPA
2022-02-20T13:53:56.351 [XNIO-1 task-1]  Kx2e2X6vTPKjQMwRTzEHJQ DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 0
2022-02-20T13:53:56.353 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 0
2022-02-20T13:53:56.354 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 4
2022-02-20T13:53:56.384 [pool-3-thread-4]   DEBUG c.n.m.k.ReactiveConsumerStartupHook$1:148 onCompletion - statusCode = 200 body  = [{"record":{"topic":"original.account","key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"partition":0,"offset":12,"headers":{"X-Correlation-Id":"n1eYKlNmQNGcBwVcefeW4g","X-Traceability-Id":"1"}},"processed":true,"correlationId":"n1eYKlNmQNGcBwVcefeW4g","traceabilityId":"1","key":"1","timestamp":1645383236342},{"record":{"topic":"original.account","key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"partition":0,"offset":13,"headers":{"X-Correlation-Id":"7wCz993MR1aAzy3sg-jkPA"}},"processed":true,"correlationId":"7wCz993MR1aAzy3sg-jkPA","traceabilityId":"2","key":"2","timestamp":1645383236355},{"record":{"topic":"original.account","value":{"accountNo":"3","firstName":"Bob","lastName":"Cat","fullName":"Bob Cat"},"partition":0,"offset":14,"headers":{"X-Correlation-Id":"wO8zExMNQ06VcvL1l3UfIw"}},"processed":true,"correlationId":"wO8zExMNQ06VcvL1l3UfIw","traceabilityId":"3","timestamp":1645383236356}]
2022-02-20T13:53:56.385 [pool-3-thread-4]   INFO  c.n.m.k.ReactiveConsumerStartupHook$1:158 onCompletion - Got successful response from the backend API

```

The log on the backend is shown as below.

```
13:53:56.258 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:67 handleRequest - Received payload at time = 2022-02-20T13:53:56.256207 with batch size = 3
13:53:56.263 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
13:53:56.270 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
13:53:56.271 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 12
13:53:56.271 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = 1
13:53:56.272 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"1","firstName":"Alex","lastName":"King"}
13:53:56.274 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = n1eYKlNmQNGcBwVcefeW4g
13:53:56.274 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Traceability-Id header value = 1
13:53:56.275 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = 1 value = {"accountNo":"1","firstName":"Alex","lastName":"King"}
13:53:56.283 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:183 produceToSidecar - Producing the transformed message = {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"n1eYKlNmQNGcBwVcefeW4g"}],"keySchemaVersion":1,"valueSchemaVersion":1} to sidecar at 2022-02-20T13:53:56.283136
13:53:56.341 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:190 produceToSidecar - Produced transformed message to sidecar at 2022-02-20T13:53:56.341578
13:53:56.342 [XNIO-1 task-1]  26iIqIcdQYGOi5wzc8e2OQ DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:126 lambda$handleRequest$1 - Producing successfully completed for 1
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 13
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = 2
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"2","firstName":"John","lastName":"Doe"}
13:53:56.343 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = 7wCz993MR1aAzy3sg-jkPA
13:53:56.343 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = 2 value = {"accountNo":"2","firstName":"John","lastName":"Doe"}
13:53:56.343 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:183 produceToSidecar - Producing the transformed message = {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"7wCz993MR1aAzy3sg-jkPA"}],"keySchemaVersion":1,"valueSchemaVersion":1} to sidecar at 2022-02-20T13:53:56.343706
13:53:56.355 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:190 produceToSidecar - Produced transformed message to sidecar at 2022-02-20T13:53:56.355500
13:53:56.355 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:126 lambda$handleRequest$1 - Producing successfully completed for 2
13:53:56.355 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
13:53:56.356 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
13:53:56.356 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 14
13:53:56.356 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = null
13:53:56.356 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
13:53:56.356 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = wO8zExMNQ06VcvL1l3UfIw
13:53:56.356 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = null value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
13:53:56.356 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:138 lambda$handleRequest$2 - Skipping produce for message with no key but value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
13:53:56.359 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:150 handleRequest - Producing done in 103 for batch size 3
13:53:56.366 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:153 handleRequest - Returning result at 2022-02-20T13:53:56.362200 batch size = 3 time taken in Millis = 109

```

### Topics

Now, let's check the original.account to see the three records we produced. Also, you see there are two records in the transformed.account topic. There is another topic called sidecar-audit that should have 3+3+2 records for produced from the curl/postman, consumed by the backend and produced by the backend. 

### Debug

If you want to understand the interactions between the Kafka-sidecar and the backend, you can set breakpoints on both sides to debug into it. It will greatly help you to understand the sidecar and help you to develop your backend application. 

### Docker

With both sidecar and backend started in the IDE, we can learn how they interact. However, these services will be deployed to the target environment with Docker-compose or Kubernetes. Here, let's see how we can start both services with docker-compose. 

In the reactive-consumer-producer example app folder, we have a docker-compose.yml file. Also, there is a config folder with sidecar and backend sub folders for each docker instance configuration. 

To start. 


```
cd ~/networknt/light-example-4j/kafka-sidecar/reactive-consumer-producer
docker-compose up
```

After sending a request from the curl or postman, you will see the logs like the following from the docker-compose console. This time, it is much more clear to see the sequence of the logs. 

```
sidecar                       | 2022-02-20T19:28:51.595 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/original.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:51.596 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/original.account
sidecar                       | 2022-02-20T19:28:51.597 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
sidecar                       | 2022-02-20T19:28:51.597 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
sidecar                       | 2022-02-20T19:28:51.607 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.config.Config$FileConfigImpl:649 loadJsonMapConfigWithSpecificConfigLoader - Trying to load mask with extension yaml, yml or json by using default loading method.
sidecar                       | 2022-02-20T19:28:51.607 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.config.Config$FileConfigImpl:508 getConfigStream - Unable to load config from externalized folder for mask.yml in /config
sidecar                       | 2022-02-20T19:28:51.608 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.config.Config$FileConfigImpl:521 getConfigStream - Trying to load config from classpath directory for file mask.yml
sidecar                       | 2022-02-20T19:28:51.608 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.config.Config$FileConfigImpl:533 getConfigStream - Config loaded from default folder for mask.yml
sidecar                       | 2022-02-20T19:28:51.609 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/original.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:51.610 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/original.account
sidecar                       | 2022-02-20T19:28:51.627 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg WARN  com.networknt.schema.JsonMetaSchema:387 newValidator - Unknown keyword components - you should define your own Meta Schema. If the keyword is irrelevant for validation, just use a NonValidationKeyword
sidecar                       | 2022-02-20T19:28:51.640 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "original.account", "original.account", topic)
sidecar                       | 2022-02-20T19:28:51.643 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.643 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.644 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.644 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:51.644 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[0])
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[1])
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records[2])
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
sidecar                       | 2022-02-20T19:28:51.645 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.646 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.646 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.646 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.646 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.646 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.647 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.647 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King"},"traceabilityId":"1"},{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe"}},{"value":{"accountNo":"3","firstName":"Bob","lastName":"Cat"}}],"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:51.647 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic original.account
sidecar                       | 2022-02-20T19:28:51.750 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 89
sidecar                       | 2022-02-20T19:28:51.751 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 1 with correlation Id 5ClrV5wrSkmF2gL7iJqFAg
sidecar                       | 2022-02-20T19:28:51.765 [XNIO-1 task-1]  HKDayK0rRK60lV_CVk8_gg DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 15
sidecar                       | 2022-02-20T19:28:51.778 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 6
sidecar                       | 2022-02-20T19:28:51.778 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 131
sidecar                       | 2022-02-20T19:28:52.523 [pool-3-thread-1]   DEBUG c.n.m.k.ReactiveConsumerStartupHook$1:125 onCompletion - polled records size = 3
sidecar                       | 2022-02-20T19:28:52.524 [pool-3-thread-1]   INFO  c.n.m.k.ReactiveConsumerStartupHook$1:132 onCompletion - Send a batch to the backend API
backend                       | 19:28:52.580 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:67 handleRequest - Received payload at time = 2022-02-20T19:28:52.578588 with batch size = 3
backend                       | 19:28:52.584 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
backend                       | 19:28:52.584 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
backend                       | 19:28:52.585 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 15
backend                       | 19:28:52.585 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = 1
backend                       | 19:28:52.586 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"1","firstName":"Alex","lastName":"King"}
backend                       | 19:28:52.597 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = 5ClrV5wrSkmF2gL7iJqFAg
backend                       | 19:28:52.598 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Traceability-Id header value = 1
backend                       | 19:28:52.598 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = 1 value = {"accountNo":"1","firstName":"Alex","lastName":"King"}
backend                       | 19:28:52.605 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:183 produceToSidecar - Producing the transformed message = {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1} to sidecar at 2022-02-20T19:28:52.605563
sidecar                       | 2022-02-20T19:28:52.642 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:52.642 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
sidecar                       | 2022-02-20T19:28:52.643 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
sidecar                       | 2022-02-20T19:28:52.643 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
sidecar                       | 2022-02-20T19:28:52.643 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:52.643 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
sidecar                       | 2022-02-20T19:28:52.644 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "transformed.account", "transformed.account", topic)
sidecar                       | 2022-02-20T19:28:52.645 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.645 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.645 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.645 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:52.645 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records[0])
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}], {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.keySchemaVersion)
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.646 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"traceabilityId":"1","correlationId":"5ClrV5wrSkmF2gL7iJqFAg"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.647 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic transformed.account
sidecar                       | 2022-02-20T19:28:52.653 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 6
sidecar                       | 2022-02-20T19:28:52.654 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 1 with correlation Id 5ClrV5wrSkmF2gL7iJqFAg
sidecar                       | 2022-02-20T19:28:52.655 [XNIO-1 task-1]  kb6mVg_ASSSy0CAX5Qusqg DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 1
sidecar                       | 2022-02-20T19:28:52.658 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 0
sidecar                       | 2022-02-20T19:28:52.658 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 11
backend                       | 19:28:52.671 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:190 produceToSidecar - Produced transformed message to sidecar at 2022-02-20T19:28:52.671809
backend                       | 19:28:52.672 [XNIO-1 task-1]  jHerO4SrSJS9s-AShizPTA DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:126 lambda$handleRequest$1 - Producing successfully completed for 1
backend                       | 19:28:52.673 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
backend                       | 19:28:52.673 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
backend                       | 19:28:52.673 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 16
backend                       | 19:28:52.673 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = 2
backend                       | 19:28:52.673 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"2","firstName":"John","lastName":"Doe"}
backend                       | 19:28:52.674 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = yeodFPpUSoKSolAcKDpf4g
backend                       | 19:28:52.674 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = 2 value = {"accountNo":"2","firstName":"John","lastName":"Doe"}
backend                       | 19:28:52.674 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:183 produceToSidecar - Producing the transformed message = {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1} to sidecar at 2022-02-20T19:28:52.674414
sidecar                       | 2022-02-20T19:28:52.675 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:52.676 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
sidecar                       | 2022-02-20T19:28:52.676 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:40 <init> - path =/producers/{topic}
sidecar                       | 2022-02-20T19:28:52.676 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:43 <init> - normalised = /producers/{topic}
sidecar                       | 2022-02-20T19:28:52.676 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:56 <init> - path = /producers/transformed.account, base path is set to: null
sidecar                       | 2022-02-20T19:28:52.677 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.openapi.ApiNormalisedPath:59 <init> - normalised = /producers/transformed.account
sidecar                       | 2022-02-20T19:28:52.677 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( "transformed.account", "transformed.account", topic)
sidecar                       | 2022-02-20T19:28:52.678 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.679 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.679 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.679 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}], {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:52.679 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records[0])
sidecar                       | 2022-02-20T19:28:52.679 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( [{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}], {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.records)
sidecar                       | 2022-02-20T19:28:52.680 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.680 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.valueSchemaVersion)
sidecar                       | 2022-02-20T19:28:52.680 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( 1, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody.keySchemaVersion)
sidecar                       | 2022-02-20T19:28:52.681 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.681 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.681 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.681 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.682 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.682 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.682 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.networknt.schema.BaseJsonValidator:149 debug - validate( {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, {"records":[{"key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"traceabilityId":"2","correlationId":"yeodFPpUSoKSolAcKDpf4g"}],"keySchemaVersion":1,"valueSchemaVersion":1}, requestBody)
sidecar                       | 2022-02-20T19:28:52.682 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg INFO  c.n.m.k.h.ProducersTopicPostHandler:117 handleRequest - ProducerTopicPostHandler handleRequest start with topic transformed.account
sidecar                       | 2022-02-20T19:28:52.683 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.m.k.h.ProducersTopicPostHandler:259 produceWithSchema - Serializing key and value with schema registry takes 0
sidecar                       | 2022-02-20T19:28:52.683 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg INFO  c.n.m.k.h.ProducersTopicPostHandler:455 produce - Associate traceability Id 2 with correlation Id yeodFPpUSoKSolAcKDpf4g
sidecar                       | 2022-02-20T19:28:52.683 [XNIO-1 task-1]  2YznCbXlSaOVYpjth0o9Tg DEBUG c.n.m.k.h.ProducersTopicPostHandler:264 produceWithSchema - Producing the entire batch to Kafka takes 0
sidecar                       | 2022-02-20T19:28:52.686 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:144 lambda$handleRequest$1 - Writing audit log takes 0
sidecar                       | 2022-02-20T19:28:52.686 [ForkJoinPool.commonPool-worker-51]   DEBUG c.n.m.k.h.ProducersTopicPostHandler:145 lambda$handleRequest$1 - ProducerTopicPostHandler handleRequest total time is 4
backend                       | 19:28:52.688 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:190 produceToSidecar - Produced transformed message to sidecar at 2022-02-20T19:28:52.687992
backend                       | 19:28:52.688 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:126 lambda$handleRequest$1 - Producing successfully completed for 2
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:72 lambda$handleRequest$2 - topic = original.account
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:74 lambda$handleRequest$2 - partition = 0
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:76 lambda$handleRequest$2 - offset = 17
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:78 lambda$handleRequest$2 - key = null
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:81 lambda$handleRequest$2 - value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
backend                       | 19:28:52.688 [XNIO-1 task-1]   INFO  c.n.m.k.b.h.KafkaRecordsPostHandler:85 lambda$handleRequest$0 - header key = X-Correlation-Id header value = O4Sk6_s0Sz6HqS6Eul4lFw
backend                       | 19:28:52.688 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:105 lambda$handleRequest$2 - Start processing of message with key = null value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
backend                       | 19:28:52.689 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:138 lambda$handleRequest$2 - Skipping produce for message with no key but value = {"accountNo":"3","firstName":"Bob","lastName":"Cat"}
backend                       | 19:28:52.691 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:150 handleRequest - Producing done in 111 for batch size 3
backend                       | 19:28:52.699 [XNIO-1 task-1]   DEBUG c.n.m.k.b.h.KafkaRecordsPostHandler:153 handleRequest - Returning result at 2022-02-20T19:28:52.694902 batch size = 3 time taken in Millis = 117
sidecar                       | 2022-02-20T19:28:52.717 [pool-3-thread-1]   DEBUG c.n.m.k.ReactiveConsumerStartupHook$1:148 onCompletion - statusCode = 200 body  = [{"record":{"topic":"original.account","key":"1","value":{"accountNo":"1","firstName":"Alex","lastName":"King","fullName":"Alex King"},"partition":0,"offset":15,"headers":{"X-Correlation-Id":"5ClrV5wrSkmF2gL7iJqFAg","X-Traceability-Id":"1"}},"processed":true,"correlationId":"5ClrV5wrSkmF2gL7iJqFAg","traceabilityId":"1","key":"1","timestamp":1645385332673},{"record":{"topic":"original.account","key":"2","value":{"accountNo":"2","firstName":"John","lastName":"Doe","fullName":"John Doe"},"partition":0,"offset":16,"headers":{"X-Correlation-Id":"yeodFPpUSoKSolAcKDpf4g"}},"processed":true,"correlationId":"yeodFPpUSoKSolAcKDpf4g","traceabilityId":"2","key":"2","timestamp":1645385332688},{"record":{"topic":"original.account","value":{"accountNo":"3","firstName":"Bob","lastName":"Cat","fullName":"Bob Cat"},"partition":0,"offset":17,"headers":{"X-Correlation-Id":"O4Sk6_s0Sz6HqS6Eul4lFw"}},"processed":true,"correlationId":"O4Sk6_s0Sz6HqS6Eul4lFw","traceabilityId":"3","timestamp":1645385332689}]
sidecar                       | 2022-02-20T19:28:52.717 [pool-3-thread-1]   INFO  c.n.m.k.ReactiveConsumerStartupHook$1:158 onCompletion - Got successful response from the backend API

```

### Tracing

Using the Kafka sidecar to produce, consumer or process streams will always enforce a traceabilityId from value to uniquely identify the entity. It will to propagated to all the services to trace the business transaction. However, the traceabilityId is not unique within the audit as the same entity might have multiple transactions within a certain timeframe. 

The Kafka sidecar will generate a correlationId and associate it with traceability to uniquely identify a transaction. This correlationId is a UUID, and it will always be unique to identify a particular transaction.

When light-portal is used, you can view the audit entries with the same correlationId to see how a message has flowed from service to service. This is very important in a data mesh or event mesh architecture. 


### Kubernetes

To be written.

### Conclusion

To be written.

### Video


{{< youtube l2mxOvGlQ1w >}}



