---
title: "Rest Query Side"
date: 2018-01-07T16:40:05-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The swagger specification for todo-list query side service can be found at
~/networknt/model-config/rest/todo-query. In the same folder, there is config.json
which is used to control how light-codegen going to generate the code.

Generate rest-query service with the following command line. Assume we are still in
light-codegen folder.

```
java -jar codegen-cli/target/codegen-cli.jar -f light-rest-4j -o ../light-example-4j/eventuate/todo-list/rest-query -m ../model-config/rest/todo_query/1.0.0/swagger.json -c ../model-config/rest/todo_query/1.0.0/config.json
``` 

Now add this rest-query into parent pom.xml and build it with maven.

```xml
    <modules>
        <module>common</module>
        <module>command</module>
        <module>query</module>
        <module>rest-command</module>
        <module>rest-query</module>
    </modules>

```

```
mvn clean install
```

The five modules should be built successfully.

Now let's update rest-query module to wire the logic.

First we need to update dependencies for this project by adding the following.
```xml
        <version.light-eventuate-4j>1.5.7</version.light-eventuate-4j>


        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>todo-common</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>todo-command</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>todo-query</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-common</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-jdbc</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-client</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>test-jdbc</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>

```

Now update the two handler as following.

```java
package com.networknt.todolist.restquery.handler;

import com.networknt.config.Config;
import com.networknt.eventuate.todolist.TodoQueryService;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class TodosGetHandler implements HttpHandler {


    TodoQueryService service =
            (TodoQueryService) SingletonServiceFactory.getBean(TodoQueryService.class);

    public void handleRequest(HttpServerExchange exchange) throws Exception {

        List<Map<String, TodoInfo>> resultAll = service.getAll();
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(resultAll));
    }
}

```


```java
package com.networknt.todolist.restquery.handler;

import com.networknt.config.Config;
import com.networknt.eventuate.todolist.TodoQueryService;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CompletableFuture;

public class TodosIdGetHandler implements HttpHandler {
    TodoQueryService service =
            (TodoQueryService) SingletonServiceFactory.getBean(TodoQueryService.class);

    public void handleRequest(HttpServerExchange exchange) throws Exception {
        String id = exchange.getQueryParameters().get("id").getFirst();
        CompletableFuture<Map<String, TodoInfo>> result = service.findById(id);
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(result));
    }
}

```



Since we are going to create some eventuate class instances from interfaces. Let's create a
service.yml under src/main/resources/config folder.

```yaml
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://localhost:3306/todo_db?useSSL=false
      username: mysqluser
      password: mysqlpw
- com.networknt.eventuate.common.impl.sync.AggregateCrud:
  - com.networknt.eventuate.jdbc.EventuateLocalAggregateStore
- com.networknt.eventuate.common.impl.AggregateEvents:
  - com.networknt.eventuate.client.KafkaAggregateSubscriptions
- com.networknt.eventuate.common.impl.AggregateCrud:
  - com.networknt.eventuate.common.impl.adapters.SyncToAsyncAggregateCrudAdapter
- com.networknt.eventuate.common.SnapshotManager:
  - com.networknt.eventuate.common.SnapshotManagerImpl
- com.networknt.eventuate.common.MissingApplyEventMethodStrategy:
  - com.networknt.eventuate.common.DefaultMissingApplyEventMethodStrategy
- com.networknt.eventuate.common.EventuateAggregateStore:
  - com.networknt.eventuate.common.impl.EventuateAggregateStoreImpl
- com.networknt.eventuate.todolist.domain.TodoAggregate:
  - com.networknt.eventuate.todolist.domain.TodoAggregate
- com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate:
  - com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate
- com.networknt.eventuate.todolist.TodoQueryRepository:
  - com.networknt.eventuate.todolist.TodoQueryRepositoryImpl
- com.networknt.eventuate.todolist.TodoQueryService:
  - com.networknt.eventuate.todolist.TodoQueryServiceImpl
- com.networknt.eventuate.todolist.TodoQueryWorkflow:
  - com.networknt.eventuate.todolist.TodoQueryWorkflow
- com.networknt.eventuate.event.EventHandlerProcessor:
  - com.networknt.eventuate.event.EventHandlerProcessorDispatchedEventReturningVoid
  - com.networknt.eventuate.event.EventHandlerProcessorDispatchedEventReturningCompletableFuture
  - com.networknt.eventuate.event.EventHandlerProcessorEventHandlerContextReturningCompletableFuture
  - com.networknt.eventuate.event.EventHandlerProcessorEventHandlerContextReturningVoid
- com.networknt.eventuate.client.SubscriptionsRegistry:
  - com.networknt.eventuate.client.SubscriptionsRegistryImpl

```


Since query side service will subscrible event from  kafka, we need create kafka config file: kafka.yml under config folder:

```yaml

description: Kafka producer and consumer properties setting
acks: all
retries: 0
batchSize: 16384
lingerms: 1
bufferMemory: 33554432
keySerializer: org.apache.kafka.common.serialization.StringSerializer
valueSerializer: org.apache.kafka.common.serialization.StringSerializer
keyDeSerializer: org.apache.kafka.common.serialization.StringDeserializer
valueDeSerializer: org.apache.kafka.common.serialization.StringDeserializer
sessionTimeout: 30000
autoOffsetreset: earliest
enableaAutocommit: false
bootstrapServers: localhost:9092

```


The rest-query will subscribe events from light-eventuate-4j so it need to
config event handler registration in the StartupHookProvider. (com.networknt.server.StartupHookProvider file under folder: src/main/resources/META-INFO.services )


```
# This is the place to plugin your startup hooks to initialize Spring application context,
# set up db connection pools and allocate resources.

# config event handle registration
com.networknt.eventuate.client.EventuateClientStartupHookProvider
```


As there are test cases and a test server will be started, we need to create a
service.yml in /src/test/resources/config folder to utilize H2 database for
our test cases. 

```yaml
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: org.h2.jdbcx.JdbcDataSource
      jdbcUrl: jdbc:h2:~/test
      username: sa
      password: sa
- com.networknt.eventuate.common.impl.sync.AggregateCrud,com.networknt.eventuate.common.impl.sync.AggregateEvents:
    - com.networknt.eventuate.test.jdbc.EventuateEmbeddedTestAggregateStore
- com.networknt.eventuate.common.impl.AggregateCrud:
  - com.networknt.eventuate.common.impl.adapters.SyncToAsyncAggregateCrudAdapter
- com.networknt.eventuate.common.impl.AggregateEvents:
  - com.networknt.eventuate.common.impl.adapters.SyncToAsyncAggregateEventsAdapter
- com.networknt.eventuate.common.SnapshotManager:
  - com.networknt.eventuate.common.SnapshotManagerImpl
- com.networknt.eventuate.common.MissingApplyEventMethodStrategy:
  - com.networknt.eventuate.common.DefaultMissingApplyEventMethodStrategy
- com.networknt.eventuate.common.EventuateAggregateStore:
  - com.networknt.eventuate.common.impl.EventuateAggregateStoreImpl
- com.networknt.eventuate.common.sync.EventuateAggregateStore:
  - com.networknt.eventuate.common.impl.sync.EventuateAggregateStoreImpl
- com.networknt.eventuate.todolist.domain.TodoAggregate:
  - com.networknt.eventuate.todolist.domain.TodoAggregate
- com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate:
  - com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate
- com.networknt.eventuate.todolist.TodoQueryRepository:
  - com.networknt.eventuate.todolist.TodoQueryRepositoryImpl
- com.networknt.eventuate.todolist.TodoQueryService:
  - com.networknt.eventuate.todolist.TodoQueryServiceImpl
- com.networknt.eventuate.todolist.TodoQueryWorkflow:
  - com.networknt.eventuate.todolist.TodoQueryWorkflow
- com.networknt.eventuate.event.EventHandlerProcessor:
  - com.networknt.eventuate.event.EventHandlerProcessorDispatchedEventReturningVoid
  - com.networknt.eventuate.event.EventHandlerProcessorDispatchedEventReturningCompletableFuture
  - com.networknt.eventuate.event.EventHandlerProcessorEventHandlerContextReturningCompletableFuture
  - com.networknt.eventuate.event.EventHandlerProcessorEventHandlerContextReturningVoid
- com.networknt.eventuate.client.SubscriptionsRegistry:
  - com.networknt.eventuate.client.SubscriptionsRegistryImpl

```


Now let's verify that all modules can be built.

```
mvn clean install
```
