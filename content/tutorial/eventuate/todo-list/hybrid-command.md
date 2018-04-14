---
title: "Hybrid Command"
date: 2018-01-07T16:40:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


So far, we have tried to implement both Command and Query services with Restful APIs. From this step, we are going to implement these service with the light-hybrid-4j framework. Although we can put cdc-service, hybrid-command, hybrid-query into the same hybrid server instance, we are going to use two hybrid server instances to
showcase Command Query Responsibility Segregation(CQRS).

If we use the hybrid framework, we don't need the RESTful CDC service anymore. We can put the cdc-service on the hybrid-command server. However, to make it simple, we are still using the Restful CDC server for this demo. 

To create command side hybrid server, we are going to use light-codegen to generate the project in todo-list. Before generating the project, we need to create a config.json to control how the generator works and a schema.json to define the service contract.

These two files can be found in model-config/hybrid/todo-command folder. 

config.json

```json
{
  "rootPackage": "com.networknt.todolist.command",
  "handlerPackage":"com.networknt.todolist.command.handler",
  "modelPackage":"com.networknt.todolist.command.model",
  "artifactId": "hybrid-command",
  "groupId": "com.networknt",
  "name": "hybrid-command",
  "version": "0.1.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8080,
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
    "name": "delete",
    "version": "0.1.0",
    "handler": "DeleteTodo",
    "scope" : "todo.w",    
    "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "id" : {
          "type" : "string"
        }
      },
      "required" : [ "id"]
    },
    "scope" : "todo.w"
  },
  {
      "name": "create",
      "version": "0.1.0",
      "handler": "CreateTodo",
      "scope" : "todo.w",
      "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "title" : {
          "type" : "string"
        },
        "completed" : {
          "type" : "boolean"
        },
        "order" : {
          "description" : "order of todo in the todo list",
          "type" : "integer",
          "minimum" : 0
        }
      },
      "required" : [ "title" ]
    },
    "scope" : "todo.w"
  },
{
      "name": "update",
      "version": "0.1.0",
      "handler": "UpdateTodo",
      "scope" : "todo.w",
      "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "id" : {
          "type" : "string"
        },
        "title" : {
          "type" : "string"
        },
        "completed" : {
          "type" : "boolean"
        },
        "order" : {
          "description" : "order of todo in the todo list",
          "type" : "integer",
          "minimum" : 0
        }
      },
      "required" : [ "id" ]
    },
    "scope" : "todo.w"
  }
  ]
}
```

Now let's run the light-codegen command line to generate the project (get latest light-codegen from master branch).

```
cd ~/networknt/light-codegen

java -jar codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o ../light-example-4j/eventuate/todo-list/hybrid-command -m ../model-config/hybrid/todo-command/schema.json -c ../model-config/hybrid/todo-command/config.json
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
    </modules>
```

```xml
            <dependency>
                <groupId>com.networknt</groupId>
                <artifactId>hybrid-command</artifactId>
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

```

With all the dependencies are added, let's change the handler to wire in the right logic.

```java
package com.networknt.todolist.command.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.networknt.config.Config;
import com.networknt.eventuate.common.AggregateRepository;
import com.networknt.eventuate.common.EventuateAggregateStore;
import com.networknt.eventuate.todolist.TodoCommandService;
import com.networknt.eventuate.todolist.TodoCommandServiceImpl;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.eventuate.todolist.domain.TodoAggregate;
import com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import java.util.concurrent.CompletableFuture;

@ServiceHandler(id="lightapi.net/todo/create/0.1.0")
public class CreateTodo implements Handler {

    private EventuateAggregateStore eventStore  = (EventuateAggregateStore) SingletonServiceFactory.getBean(EventuateAggregateStore.class);

    private AggregateRepository todoRepository = new AggregateRepository(TodoAggregate.class, eventStore);
    private AggregateRepository bulkDeleteAggregateRepository  = new AggregateRepository(TodoBulkDeleteAggregate.class, eventStore);

    private TodoCommandService service = new TodoCommandServiceImpl(todoRepository, bulkDeleteAggregateRepository);

    @Override
    public ByteBuffer handle(Object input)  {

        JsonNode inputPara = Config.getInstance().getMapper().valueToTree(input);

        TodoInfo todo = new TodoInfo();
        todo.setTitle(inputPara.findPath("title").asText());
        todo.setCompleted(inputPara.findPath("completed").asBoolean());
        todo.setOrder(inputPara.findPath("order").asInt());

        CompletableFuture<TodoInfo> result = service.add(todo).thenApply((e) -> {
            TodoInfo m = e.getAggregate().getTodo();
            return m;
        });

        return NioUtils.toByteBuffer("{\"message\":" + result + "}");
    }
}

```

```java
package com.networknt.todolist.command.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.networknt.config.Config;
import com.networknt.eventuate.common.AggregateRepository;
import com.networknt.eventuate.common.EventuateAggregateStore;
import com.networknt.eventuate.todolist.TodoCommandService;
import com.networknt.eventuate.todolist.TodoCommandServiceImpl;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.eventuate.todolist.domain.TodoAggregate;
import com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import java.util.concurrent.CompletableFuture;

@ServiceHandler(id="lightapi.net/todo/delete/0.1.0")
public class DeleteTodo implements Handler {

    private EventuateAggregateStore eventStore  = (EventuateAggregateStore) SingletonServiceFactory.getBean(EventuateAggregateStore.class);

    private AggregateRepository todoRepository = new AggregateRepository(TodoAggregate.class, eventStore);
    private AggregateRepository bulkDeleteAggregateRepository  = new AggregateRepository(TodoBulkDeleteAggregate.class, eventStore);

    private TodoCommandService service = new TodoCommandServiceImpl(todoRepository, bulkDeleteAggregateRepository);


    @Override
    public ByteBuffer handle(Object input)  {

        JsonNode inputPara = Config.getInstance().getMapper().valueToTree(input);
        // delete a todo-event
        String id = inputPara.findPath("id").asText();

        CompletableFuture<TodoInfo> result = service.remove(id).thenApply((e) -> {
            TodoInfo m = e.getAggregate().getTodo();
            return m;
        });


        return NioUtils.toByteBuffer("{\"message\":" + result + "}");
    }
}

```

```java
package com.networknt.todolist.command.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.networknt.config.Config;
import com.networknt.eventuate.common.AggregateRepository;
import com.networknt.eventuate.common.EventuateAggregateStore;
import com.networknt.eventuate.todolist.TodoCommandService;
import com.networknt.eventuate.todolist.TodoCommandServiceImpl;
import com.networknt.eventuate.todolist.common.model.TodoInfo;
import com.networknt.eventuate.todolist.domain.TodoAggregate;
import com.networknt.eventuate.todolist.domain.TodoBulkDeleteAggregate;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import java.util.concurrent.CompletableFuture;

@ServiceHandler(id="lightapi.net/todo/update/0.1.0")
public class UpdateTodo implements Handler {

    private EventuateAggregateStore eventStore  = (EventuateAggregateStore) SingletonServiceFactory.getBean(EventuateAggregateStore.class);

    private AggregateRepository todoRepository = new AggregateRepository(TodoAggregate.class, eventStore);
    private AggregateRepository bulkDeleteAggregateRepository  = new AggregateRepository(TodoBulkDeleteAggregate.class, eventStore);

    private TodoCommandService service = new TodoCommandServiceImpl(todoRepository, bulkDeleteAggregateRepository);
    @Override
    public ByteBuffer handle(Object input)  {

        JsonNode inputPara = Config.getInstance().getMapper().valueToTree(input);

        String id = inputPara.findPath("id").asText();;
        TodoInfo todo = new TodoInfo();
        todo.setTitle(inputPara.findPath("title").asText());
        todo.setCompleted(inputPara.findPath("completed").asBoolean());
        todo.setOrder(inputPara.findPath("order").asInt());

        CompletableFuture<TodoInfo> result = service.update(id, todo).thenApply((e) -> {
            TodoInfo m = e.getAggregate().getTodo();
            return m;
        });


        return NioUtils.toByteBuffer("{\"message\":" + result + "}");
    }
}

```

As we have to put the service jar into the hybrid server platform in order to run it, we cannot wait then to test it. So for hybrid service development, the end-to-end test is very important to ensure what you have built is working.

```java
package com.networknt.todolist.command.handler;

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

public class CreateTodoTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(CreateTodo.class);

    @Test
    public void testCreateTodo() throws ClientException, ApiException {
        CloseableHttpClient client = Client.getInstance().getSyncClient();
        HttpPost httpPost = new HttpPost("http://localhost:8080/api/json");

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("host", "lightapi.net");
        map.put("service", "todo");
        map.put("action", "create");
        map.put("version", "0.1.0");
        map.put("title", "create todo from hybrid service unit test");
        map.put("completed", false);
        map.put("order", 1);
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
package com.networknt.todolist.command.handler;

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

public class DeleteTodoTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(DeleteTodo.class);

    @Test
    public void testDeleteTodo() throws ClientException, ApiException {
        CloseableHttpClient client = Client.getInstance().getSyncClient();
        HttpPost httpPost = new HttpPost("http://localhost:8080/api/json");
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("host", "lightapi.net");
        map.put("service", "todo");
        map.put("action", "delete");
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

```java
package com.networknt.todolist.command.handler;

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

public class UpdateTodoTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(UpdateTodo.class);

    @Test
    public void testUpdateTodo() throws ClientException, ApiException {
        CloseableHttpClient client = Client.getInstance().getSyncClient();
        HttpPost httpPost = new HttpPost("http://localhost:8080/api/json");

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("host", "lightapi.net");
        map.put("service", "todo");
        map.put("action", "update");
        map.put("version", "0.1.0");
        map.put("id", "101010");
        map.put("title", "create todo from hybrid service unit test");
        map.put("completed", false);
        map.put("order", 1);
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

In order to make the test case works, we need to add a service.yml in
src/test/resources/config folder

```yaml
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      jdbcUrl: jdbc:mysql://localhost:3306/todo_db?useSSL=false
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
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
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

Before running the tests, we need to add the following test dependencies into the pom.xml 

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

In the next step, we are going to create a [hybrid query][] side service with the light-hybrid-4j framework. 

[hybrid query]: /tutorial/eventuate/todo-list/hybrid-query/
