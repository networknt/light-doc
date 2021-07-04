---
title: "User Query with ksqlDB"
date: 2021-01-17T13:50:12-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this tutorial, we will re-implement the stream-query with ksqldb instead of Kafka Streams. 

### Codegen

The README.md and config.json to scaffold a project in the light-example-4j can be found at the following URL. 

https://github.com/networknt/model-config/tree/master/kafka/ksqldb-query

The generated project can be found at

https://github.com/networknt/light-example-4j/tree/master/kafka/ksqldb-query

### Start Confluent

Download and install the confluent 6.0.1 locally with tar.gz file and start the services

```
confluent local services start
```

The ksqlDB server will be started along with other services. 


### Producer

Please refer to the [producer-consumer][] to create the test topic and produce some message to the test topic. 

### Web Console

Access the web console from http://localhost:9021/ and go to the ksqlDB Editor. Issue the following command to create a table. 


```
CREATE TABLE USER_TABLE (userId VARCHAR PRIMARY KEY, firstName VARCHAR, lastName VARCHAR, country VARCHAR)
    WITH (kafka_topic='test', value_format='json');

```

Once the table is created, we can issue the following command to query the table. 

```
SELECT * from  USER_TABLE WHERE  USERID = 'stevehu' EMIT CHANGES;
```

As we have run the producer before, the messages are already sent to the test topic. We need to click the Add query properties link to add the following property.

auto.offset.reset = Earliest

Click the Run query button, and you will see the result like the following.

```
[{"USERID":"stevehu","FIRSTNAME":"Steve","LASTNAME":"Hu","COUNTRY":"CA"}]
```

### pom.xml

In order to access the ksqlDB from our service, we need to add client dependency. 

```
        <version.ksqldb>0.14.0</version.ksqldb>

        <dependency>
            <groupId>io.confluent.ksql</groupId>
            <artifactId>ksqldb-api-client</artifactId>
            <version>${version.ksqldb}</version>
        </dependency>

        <repository>
            <id>ksqlDB</id>
            <name>ksqlDB</name>
            <url>https://ksqldb-maven.s3.amazonaws.com/maven/</url>
        </repository>
        <repository>
            <id>confluent</id>
            <name>Confluent</name>
            <url>https://jenkins-confluent-packages-beta-maven.s3.amazonaws.com/6.1.0-beta201006024150/1/maven/</url>
        </repository>
    <pluginRepositories>
        <pluginRepository>
            <id>ksqlDB</id>
            <url>https://ksqldb-maven.s3.amazonaws.com/maven/</url>
        </pluginRepository>
        <pluginRepository>
            <id>confluent</id>
            <url>https://jenkins-confluent-packages-beta-maven.s3.amazonaws.com/6.1.0-beta201006024150/1/maven/</url>
        </pluginRepository>
    </pluginRepositories>
                    
```

### QueryUserIdGetHandler

The following is the handler. 

```
package com.networknt.kafka.handler;

import com.networknt.handler.LightHttpHandler;
import io.confluent.ksql.api.client.*;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Headers;
import io.undertow.util.HttpString;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.Map;

/**
For more information on how to write business handlers, please check the link below.
https://doc.networknt.com/development/business-handler/rest/
*/
public class QueryUserIdGetHandler implements LightHttpHandler {
    private static final Logger logger = LoggerFactory.getLogger(QueryUserIdGetHandler.class);
    private static final String QUERY = "SELECT * from USER_TABLE WHERE USERID = '%s' EMIT CHANGES;";
    private static final String OBJECT_NOT_FOUND = "ERR11637";
    private static String KSQLDB_SERVER_HOST = "localhost";
    private static int KSQLDB_SERVER_HOST_PORT = 8088;
    public Client client = null;

    public QueryUserIdGetHandler() {
        ClientOptions options = ClientOptions.create()
                .setHost(KSQLDB_SERVER_HOST)
                .setPort(KSQLDB_SERVER_HOST_PORT);
        client = Client.create(options);
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        String userId = exchange.getQueryParameters().get("userId").getFirst();
        if(logger.isTraceEnabled()) logger.trace("userId = " + userId);
        String s = String.format(QUERY, userId);
        if(logger.isTraceEnabled()) logger.trace("query = " + s);
        Map<String, Object> properties = new HashMap<>();
        properties.put("auto.offset.reset", "earliest");
        StreamedQueryResult result = client.streamQuery(s, properties).get();
        Row row = result.poll();
        if(row != null) {
            exchange.getResponseHeaders().add(Headers.CONTENT_TYPE, "application/json");
            KsqlArray a = row.values();
            exchange.getResponseSender().send(a.toJsonString());
        } else {
            setExchangeStatus(exchange, OBJECT_NOT_FOUND, "user", userId);
        }
    }
}

```

### Test

Issue the following command from the terminal to test it. 

```
curl -k https://localhost:8443/query/stevehu
```

And the result:

```
["stevehu","Steve","Hu","CA"]
```

### Summary

As you can see, the result is a list of data elements in the table. It is due to the design by the ksqldb client to mimic the database query result. In the return row object, we have access to each column's header and data type. It is easy to convert the result to a JSON if we want. 

Compared with the Kafka [stream-query][] implementation; the service is significantly slower. It will get slower when more user objects are sent to the test topic because we set the property to redo the stream from the earliest each query dynamically. 

In essence, we delegate the Streams processing to the ksqlDB server with SQL like statements, and then we call the ksqlDB with its Java client from our service built with light-rest-4j. In the stream-query, we combine the Streams processing and query on the local store in the same service to gain the best performance and reduce the license cost for the ksqlDB. 


[producer-consumer]: /tutorial/light-kafka/producer-consumer/
[stream-query]: /tutorial/light-kafka/stream-query/
