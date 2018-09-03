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
reviewed: true
---

The swagger specification for the todo-list query side service can be found at ~/networknt/model-config/rest/todo-query. In the same folder, there is config.json which is used to control how light-codegen is going to generate the code.

Generate rest-query service with the following command line. Assume we are still in light-codegen folder.

```
java -jar codegen-cli/target/codegen-cli.jar -f swagger -o ../light-example-4j/eventuate/todo-list/rest-query -m ../model-config/rest/todo_query/1.0.0/swagger.json -c ../model-config/rest/todo_query/1.0.0/config.json
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

Now let's update rest-query module to wire in the logic.

First, we need to update dependencies for this project by adding the following.

```xml
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

Now update the two handlers as following.

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



Since we are going to create some eventuate class instances from interfaces, let's create a service.yml under src/main/resources/config folder.

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


Since query side service subscribes the events from  Kafka, we need to create a kafka.yml under config folder:

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


The rest-query subscribes events from light-eventuate-4j, so it needs to config event handler registration in the StartupHookProvider. The provider is defined in service.yml file like below.

```
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.eventuate.client.EventuateClientStartupHookProvider
```


As there are test cases and a test server will be started, we need to create a
service.yml in /src/test/resources/config folder to utilize H2 database for
our test cases. 

```yaml
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      jdbcUrl: jdbc:mysql://198.55.49.187:3306/todo_db?useSSL=false
      username: mysqluser
      password: mysqlpw
      maximumPoolSize: 15
      useServerPrepStmts: true
      cachePrepStmts: true
      cacheCallableStmts: true
      prepStmtCacheSize: 4096
      prepStmtCacheSqlLimit: 2048
- com.networknt.eventuate.jdbc.EventuateJdbcAccess:
  - com.networknt.eventuate.jdbc.EventuateJdbcAccessImpl
- com.networknt.eventuate.common.impl.sync.AggregateCrud:
  - com.networknt.eventuate.server.jdbckafkastore.EventuateLocalAggregateCrud
- com.networknt.eventuate.common.impl.AggregateCrud:
  - com.networknt.eventuate.common.impl.adapters.SyncToAsyncAggregateCrudAdapter
- com.networknt.eventuate.common.impl.AggregateEvents:
  - com.networknt.eventuate.server.jdbckafkastore.EventuateKafkaAggregateSubscriptions
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
- com.networknt.eventuate.client.SubscriptionsRegistry: com.networknt.eventuate.client.EventuateClientServiceInitializer::subscriptionsRegistry

# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.todolist.restquery.PathHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.eventuate.client.EventuateClientStartupHookProvider
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Swagger Parsing swagger specification based on request uri and method.
  - com.networknt.swagger.SwaggerHandler
  # Security JWT token verification and scope verification (depending on SwaggerHandler)
  - com.networknt.security.JwtVerifyHandler
  # Body Parse body based on content type in the header.
  - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on swagger specification (depending on Swagger and Body)
  - com.networknt.validator.ValidatorHandler

```


Now let's verify that all modules can be built.

```
mvn clean install
```

If all modules can be built successfully, we can move to the next step to [test rest][] services. 

[test rest]: /tutorial/eventuate/todo-list/rest-test/
