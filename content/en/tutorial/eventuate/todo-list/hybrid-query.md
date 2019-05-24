---
title: "Hybrid Query"
date: 2018-01-07T16:40:21-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


To create query side hybrid service, we are going to use light-codegen to generate the project in todo-list. Before generating the project, we need to create a config.json to control how the generator works and a schema.json to define the service contract.

These two files can be found in model-config/hybrid/todo-query folder. 

config.json

```json
{
  "rootPackage": "com.networknt.todolist.query",
  "handlerPackage":"com.networknt.todolist.query.handler",
  "modelPackage":"com.networknt.todolist.query.model",
  "artifactId": "hybrid-query",
  "groupId": "com.networknt",
  "name": "hybrid-query",
  "version": "0.1.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8082,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": false,
  "supportPostgresql": false,
  "supportH2ForTest": false,
  "supportClient": false,
  "dockerOrganization": "networknt"
}
```

schema.json 

```json
{
  "host": "lightapi.net",
  "service": "todo",
  "action": [
 {
    "name": "gettodo",
    "version": "0.1.0",
    "handler": "GetTodoById",
    "scope" : "todo.r", 
    "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "id" : {
          "type" : "string"
        }
      },
      "required" : [ "id" ]
    },
    "scope" : "todo.r"
  },
 {
     "name": "gettodos",
     "version": "0.1.0",
     "handler": "GetAllTodos",
    "scope" : "todo.r", 
    "schema" : {
      "title" : "Service"
    },
    "scope" : "todo.r"
  }
  ]
}
```

Now let's run the light-codegen command line to generate the project.

```
cd ~/networknt/light-codegen

java -jar codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o ../light-example-4j/eventuate/todo-list/hybrid-query -m ../model-config/hybrid/todo-query/schema.json -c ../model-config/hybrid/todo-query/config.json
```

Let's add this newly generated project to the parent pom.xml and build all modules.

```xml
    <modules>
        <module>common</module>
        <module>command</module>
        <module>query</module>
        <module>rest-command</module>
        <module>rest-query</module>
        <module>hybrid-command</module>
        <module>hybrid-query</module>
    </modules>
```

```xml
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>hybrid-query</artifactId>
                <version>${project.version}</version>
            </dependency>

```

Rebuild all modules from the root folder.

```
mvn clean install

```

Now let's change the dependencies to add light-eventuate-4j modules and todo-list modules.


```xml
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

```

With all the dependencies are added, let's change the handler to wire in the right logic.

```java
package com.networknt.todolist.query.handler;

import com.networknt.config.Config;
import com.networknt.eventuate.todolist.TodoQueryService;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import java.util.List;
import java.util.Map;

@ServiceHandler(id="lightapi.net/todo/gettodos/0.1.0")
public class GetAllTodos implements Handler {
    TodoQueryService service =
            (TodoQueryService) SingletonServiceFactory.getBean(TodoQueryService.class);

    @Override
    public ByteBuffer handle(Object input)  {

        List<Map<String, TodoInfo>> resultAll = service.getAll();
        String returnMessage = null;
        try {
            returnMessage = Config.getInstance().getMapper().writeValueAsString(resultAll);
        } catch ( Exception e) {

        }
        return NioUtils.toByteBuffer("{\"message\":" +returnMessage + "}");

    }
}

```

```java
package com.networknt.todolist.query.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.networknt.config.Config;
import com.networknt.eventuate.todolist.TodoQueryService;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import java.util.Map;
import java.util.concurrent.CompletableFuture;

@ServiceHandler(id="lightapi.net/todo/gettodo/0.1.0")
public class GetTodoById implements Handler {
    TodoQueryService service =
            (TodoQueryService) SingletonServiceFactory.getBean(TodoQueryService.class);

    @Override
    public ByteBuffer handle(Object input) {

        JsonNode inputPara = Config.getInstance().getMapper().valueToTree(input);

        String id = inputPara.findPath("id").asText();


        CompletableFuture<Map<String, TodoInfo>> result = service.findById(id);
        String returnMessage = null;
        try {
            returnMessage = Config.getInstance().getMapper().writeValueAsString(result);
        } catch (Exception e) {

        }
        return NioUtils.toByteBuffer("{\"message\":" + returnMessage + "}");
    }
}

```

As we have to put the service jar into the hybrid server platform in order to run it, we cannot wait then to test it. So for hybrid service development, the end-to-end test is very important to ensure what you have built is working.

Here are the two tests.

```java
package com.networknt.todolist.query.handler;

import com.networknt.client.Client;
import com.networknt.server.Server;
import com.networknt.exception.ClientException;
import com.networknt.exception.ApiException;
import org.apache.commons.io.IOUtils;
import org.apache.http.HttpResponse;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.ResponseHandler;
import org.apache.http.client.methods.*;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.jose4j.json.internal.json_simple.JSONObject;
import org.junit.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class GetAllTodosTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(GetAllTodos.class);

    @Test
    public void testGetAllTodos() throws ClientException, ApiException {
        CloseableHttpClient client = Client.getInstance().getSyncClient();
        HttpPost httpPost = new HttpPost("http://localhost:8082/api/json");
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("host", "lightapi.net");
        map.put("service", "todo");
        map.put("action", "gettodos");
        map.put("version", "0.1.0");

        JSONObject json = new JSONObject();
        json.putAll( map );
        System.out.printf( "JSON: %s", json.toString() );
        //Client.getInstance().addAuthorization(httpPost);
        try {
            httpPost.setEntity(new StringEntity(json.toString()));
            httpPost.setHeader("Content-type", "application/json");
            CloseableHttpResponse response = client.execute(httpPost);
            Assert.assertEquals(200, response.getStatusLine().getStatusCode());
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

```java
package com.networknt.todolist.query.handler;

import com.networknt.client.Client;
import com.networknt.server.Server;
import com.networknt.exception.ClientException;
import com.networknt.exception.ApiException;
import org.apache.commons.io.IOUtils;
import org.apache.http.HttpResponse;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.ResponseHandler;
import org.apache.http.client.methods.*;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.jose4j.json.internal.json_simple.JSONObject;
import org.junit.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class GetTodoByIdTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(GetTodoById.class);

    @Test
    public void testGetTodoById() throws ClientException, ApiException {
        CloseableHttpClient client = Client.getInstance().getSyncClient();
        HttpPost httpPost = new HttpPost("http://localhost:8082/api/json");
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("host", "lightapi.net");
        map.put("service", "todo");
        map.put("action", "gettodo");
        map.put("version", "0.1.0");
        map.put("id", "101010");

        JSONObject json = new JSONObject();
        json.putAll( map );
        System.out.printf( "JSON: %s", json.toString() );

        try {
            httpPost.setEntity(new StringEntity(json.toString()));
            httpPost.setHeader("Content-type", "application/json");
            CloseableHttpResponse response = client.execute(httpPost);
            Assert.assertEquals(200, response.getStatusLine().getStatusCode());
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

The test server will load some light-eventuate-4j classes and H2 database and it is defined in service.yml in src/test/resources/config folder.

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
- com.networknt.eventuate.common.impl.sync.AggregateEvents:
  - com.networknt.eventuate.client.EventuateLocalAggregatesEvents
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
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.rpc.router.RpcRouter
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.rpc.router.RpcStartupHookProvider
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Security JWT token verification and scope verification (depending on SwaggerHandler)
  - com.networknt.rpc.security.JwtVerifyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler

```

Before running the tests, we need to add the following test dependencies into pom.xml 

```xml
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>${version.hikaricp}</version>
        </dependency>

        <!-- Test Dependencies -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${version.h2}</version>
            <scope>test</scope>
        </dependency>

```

Now you can build the entire project from the root folder of the todo-list project.

```
mvn clean install
``` 

In the next step, we are going to [test hybrid services][] we just built.  

[test hybrid services]: /tutorial/eventuate/todo-list/hybrid-test/
