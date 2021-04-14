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

The reactive consumer is similar to the Kafka Streams. The backend API/APP exposes an API endpoint to receive data once the sidecar detects new messages in the Kafka topic(s). 

### Configuration

The config file kafka-consumer.yml is shared by both the active consumer and the reactive consumer. But there are several properties only for the reactive consumer. 

```
# Reactive Consumer Specific Configuration
# The topic that is going to be consumed. For reactive consumer only in the kafka-sidecar.
# If two or more topics are going to be subscribed, concat them with comma without space.
topic: test1,test2
# The default consumer group. For reactive consumer only in the kafka-sidecar.
groupId: group1
# Waiting period in millisecond to poll another batch
waitPeriod: 1000
```

Here we define the consumer group, the topic or topics to be consumed and the waiting period between different polls to the Kafka topic(s).

### Backend API Specification

As the sidecar will invoke an API to the backend API once the data is available, the backend API must have an endpoint defined with the following OpenAPI 3.0 specification. 

```
---
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
        "201":
          description: "Null response"
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
          type: "string"
        value:
          type: "string"
        header:
          type: "object"
        partition:
          type: "integer"
          minimum: 0
        offset:
          type: "number"
          minimum: 0

```

The path of the endpoing is configurable on the kafka-sidecar, so you can customize the endpoint for your backend API implementation. 


### Backend API Implementation


The following is a simple light-4j handler implementation to output he received messages to the console via logging statements. 

```
package com.networknt.mesh.kafka.backend.handler;

import com.networknt.body.BodyHandler;
import com.networknt.handler.LightHttpHandler;
import io.undertow.server.HttpServerExchange;
import java.util.List;
import java.util.Map;

/**
For more information on how to write business handlers, please check the link below.
https://doc.networknt.com/development/business-handler/rest/
*/
public class KafkaRecordsPostHandler implements LightHttpHandler {

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<Map<String, Object>> records = (List<Map<String, Object>>)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        for(int i = 0; i < records.size(); i++) {
            Map<String, Object> record = records.get(i);
            logger.debug("topic = " + record.get("topic"));
            logger.debug("partition = " + record.get("partition"));
            logger.debug("offset = " + record.get("offset"));
            logger.debug("key = " + record.get("key"));
            logger.debug("value = " + record.get("value"));
            Map<String, String> headerMap = (Map<String, String>)record.get("headers");
            headerMap.entrySet().forEach(entry -> {
                logger.debug("header key = " + entry.getKey() + " header value = " + entry.getValue());
            });
        }
        exchange.setStatusCode(201);
        exchange.endExchange();
    }
}

```

The example application can be found on the Github.com in /networknt/light-example-4j/kafka/sidecar-backend/

### Test Within IDE


### Test with Docker


### Test in Kubernetes




