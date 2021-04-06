---
title: "Ksqldb Consumer"
date: 2021-04-03T21:29:55-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Unlike other consumers, the KsqlDB consumer is connecting to the KsqlDB instead of the Kafka cluster. Because the KsqlDB has processed the streams on the server, the Kafka-sidecar is just a client for the KsqlDB. 

### Configuration

The following is the config file kafka-ksqldb.yml

```
# ksqlDB host
host: localhost
# ksqlDB port
port: 8088
# query for the ksqldb. It can be a table or stream.
query: SELECT * from TEST EMIT CHANGES;
# stream query properties
properties:
  auto.offset.reset: earliest

```


To enable the consumer, we also need to update the service.yml file to add the startup and shutdown hooks. 

```
- com.networknt.server.StartupHookProvider:
  - com.networknt.mesh.kafka.KsqldbConsumerStartupHook
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.mesh.kafka.KsqldbConsumerShutdownHook

```

### Backend API Specification

The following is one of the examples for the backend API to receive the call from the Kafka sidecar. The path is configurable from the sidecar. 

---
openapi: "3.0.0"
info:
  version: "1.0.0"
  title: "KsqlDB Backend"
  license:
    name: "MIT"
servers:
- url: "http://backend.ksqldb.io"
paths:
  /kafka/ksqldb:
    post:
      summary: "Create a user row"
      operationId: "createUserRow"
      requestBody:
        description: "An array that represent a user row"
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/UserRow"
      security:
      - ksqldb_auth:
        - "write:ksqldb"
      responses:
        "201":
          description: "Null response"
components:
  securitySchemes:
    ksqldb_auth:
      type: "oauth2"
      description: "This API uses OAuth 2 with the client credential grant flow."
      flows:
        clientCredentials:
          tokenUrl: "https://localhost:6882/token"
          scopes:
            write:ksqldb: "modify user row"
  schemas:
    UserRow:
      type: "array"
      items:
        description: "user row data"
        type: "string"


### Backend API Implementation

Here is an example implementation in light-4j and it is a post handler. 

```
package com.networknt.kafka.handler;

import com.networknt.body.BodyHandler;
import com.networknt.config.JsonMapper;
import com.networknt.handler.LightHttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

/**
For more information on how to write business handlers, please check the link below.
https://doc.networknt.com/development/business-handler/rest/
*/
public class KsqldbPostHandler implements LightHttpHandler {
    private static final Logger logger = LoggerFactory.getLogger(KsqldbPostHandler.class);

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        List<String> body = (List<String>)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        logger.info("received call from the sidecar: " + JsonMapper.toJson(body));
        exchange.setStatusCode(201);
        exchange.endExchange();
    }
}

```

The received data from the sidecar is logged with logback and a status code 204 no content is returned to the sidecar to indicate that the process is done successfully. 

The example can be found in the [light-example-4j/kafka/ksqldb-backend](https://github.com/networknt/light-example-4j/tree/release/kafka/ksqldb-backend)

