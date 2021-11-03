---
title: "Ksqldb Active Consumer Tutorial"
date: 2021-09-05T11:48:10-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


### Summary

Pull queries are a relatively new but integral feature offered by ksqlDB. In contrast to push queries, which perpetually stream incremental query results to clients, pull queries follow a traditional request/response model, which means that a pull query retrieves a finite result from the ksqlDB server and terminate on completion, similar to the way queries work with traditional databases.
KsqlDB server is part of the Confluent Platform, and it provides the streams processor to hide the details of the Kafka streams. Users can create tables and streams on the KsqlDB server and subscribe to the KsqlDB server from the sidecar. Once the table or stream is changed, a rest call will be issued by the sidecar to the backend API to notify that new data received. 

There are two types of queries in ksqldb:

- pull
- push

Please refer [here](https://docs.ksqldb.io/en/latest/concepts/queries/) for detail.

"/ksqldb/active" endpoint support both query types, but we suggest to use pull query for this endpoint only. It much stable and have better performance.

- Pull queries are expressed using a strict subset of ANSI SQL.
- You can issue a pull query against any table that was created by a CREATE TABLE AS SELECT statement.
- Currently, we do not support pull queries against tables created by using a CREATE TABLE statement.
- Pull queries do not support JOIN, PARTITION BY, GROUP BY and WINDOW clauses (but can query materialized tables that contain those clauses).


If you query to against KStream or KTable which  created by using a CREATE TABLE statement, set the query type as "push".


There is new request object has been added into light-kafka:

https://github.com/networknt/light-kafka/blob/master/kafka-entity/src/main/java/com/networknt/kafka/entity/KsqlDbPullQueryRequest.java

Sample request payload:

```text
{
    "offset": "earliest",
    "deserializationError": false,
    "queryType": "pull",
     "tableScanEnable": true,
    "query": "select * from QUERYUSER;"
}

```
#### Fields detail:

- offset
  optional field, only use for push query. Available values: earliest/latest

- queryType
  optional field, indicate query type. Available values: pull/push

- deserializationError
  optional field, indicates whether to fail if corrupt messages are read. Available values: true/false

- tableScanEnable
  optional field, indicates whether full table scan allowed. Available values: true/false

- query
  required field, ksqlDB query string

-----

In light-4j kafka-sidecar, there is startuphook (KsqldbActiveConsumerStartupHook) which use to initial ksql Active consumer.

KsqldbActiveConsumerStartupHook will initial a kafka API client based the kafka-ksqldb.yml config.

For local connection, it only need host and port for connection:

```text
ksqldbHost: ${kafka-ksqldb.ksqldbHost:localhost}
# ksqlDB port
ksqldbPort: ${kafka-ksqldb.ksqldbPort:8088}
```

For Enterprise Kafka KSQL server, we need use tls connection and use the base Authentication:

```text
useTls: ${kafka-ksqldb.useTls:false}
trustStore: ${kafka-ksqldb.trustStore:/truststore/kafka.server.truststore.jks}
trustStorePassword: ${kafka-ksqldb.trustStorePassword:changeme}
basicAuthCredentialsUser: ${kafka-ksqldb.basicAuthCredentialsUser:userId}
basicAuthCredentialsPassword: ${kafka-ksqldb.basicAuthCredentialsPassword:changeme}
```

There is an API endpoint for user to use Active consumer to run query and get the query result as API response:

There is new endpoint added for executing ksqlDB query:


```text
  '/ksqldb/active':
    post:
      operationId: KsqlDBPullQueryActive
      summary: KsqlDBPullQuery APIs by active consumer
      requestBody:
        description: "process a ksqlDB query"
        required: true
        content:
          application/json:
            schema:
              "$ref": "#/components/schemas/KsqlDbPullQueryRequest"
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
```


#### Verify kafka sidecar ksql query locally :

- Create a topic name as test:

![create-topic](/images/create-topic.png)

And set the value JSON schema:

```json
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

And then use the kafka sidecar "/producers/test" endpoint (POST) to populate some message to the topic.

Sample request body:

```json
{
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
}
```

- Create KTable based on the topic created above:

```json
CREATE TABLE USERS 
   (ID STRING PRIMARY KEY, USERID STRING, FIRSTNAME STRING, LASTNAME STRING, COUNTRY STRING) 
    WITH (KAFKA_TOPIC='test', KEY_FORMAT='KAFKA', VALUE_FORMAT='JSON_SR');
```

- Create query able KTable based on the KTable above:

```json

 CREATE TABLE QUERYUSER AS SELECT * FROM USERS;

```

- Start kafka sidecar and verify by curl command:

```text
curl --location --request POST 'http://localhost:8084/ksqldb/active' \
--header 'Content-Type: application/json' \
--data-raw ' 
{
    "offset": "earliest",
    "deserializationError": false,
    "queryType": "pull",
     "tableScanEnable": true,
    "query": "select * from QUERYUSER1 where id = '\''1'\'';"
}
'
```

Response:

```text
[
    {
        "USERID": "4444",
        "FIRSTNAME": "test1",
        "ID": "1"
    }
]
```