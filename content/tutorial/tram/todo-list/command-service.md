---
title: "Command Service"
date: 2018-01-08T10:04:10-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This is a restful microservice generated from config.json and OpenAPI 3.0 specification
locate at https://github.com/networknt/model-config/tree/master/rest/tram-todo/multi-module/command-service

Regarding to light-codegen generator, please refer to [light-codegen tutorial][].

Here we are going to walk through the handler implementations.

TodosPostHandler is used to create a new Todo object.

```java

package com.networknt.tram.todolist.command.handler;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.body.BodyHandler;
import com.networknt.config.Config;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.todolist.command.CreateTodoRequest;
import com.networknt.tram.todolist.command.Todo;
import com.networknt.tram.todolist.command.TodoCommandService;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class TodosPostHandler implements HttpHandler {
    private TodoCommandService todoCommandService = (TodoCommandService) SingletonServiceFactory.getBean(TodoCommandService.class);

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {

        ObjectMapper mapper = new ObjectMapper();

        // add a new object
        Map s = (Map)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        String json = mapper.writeValueAsString(s);
        CreateTodoRequest createTodoRequest = mapper.readValue(json, CreateTodoRequest.class);
        Todo todo = todoCommandService.create(createTodoRequest);

        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(todo));

    }
}

```

TodosIdPutHandler is used to update existing Todo object.

```java

package com.networknt.tram.todolist.command.handler;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.body.BodyHandler;
import com.networknt.config.Config;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.todolist.command.Todo;
import com.networknt.tram.todolist.command.TodoCommandService;
import com.networknt.tram.todolist.command.TodoNotFoundException;
import com.networknt.tram.todolist.command.UpdateTodoRequest;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class TodosIdPutHandler implements HttpHandler {
    private TodoCommandService todoCommandService = (TodoCommandService) SingletonServiceFactory.getBean(TodoCommandService.class);
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {

        String id = exchange.getQueryParameters().get("id").getFirst();

        ObjectMapper mapper = new ObjectMapper();

        // add a new object
        Map s = (Map)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        String json = mapper.writeValueAsString(s);
        UpdateTodoRequest updateTodoRequest = mapper.readValue(json, UpdateTodoRequest.class);

        try {
            Todo todo = todoCommandService.update(id, updateTodoRequest);
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
            exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(todo));
        } catch ( TodoNotFoundException e) {
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
            exchange.getResponseSender().send("No record been updated");
        }
    }
}

```

TodosIdDeleteHandler is used to delete a Todo object.

```java
package com.networknt.tram.todolist.command.handler;

import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.todolist.command.TodoCommandService;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class TodosIdDeleteHandler implements HttpHandler {

    private TodoCommandService todoCommandService = (TodoCommandService) SingletonServiceFactory.getBean(TodoCommandService.class);
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {


        String id = exchange.getQueryParameters().get("id").getFirst();
        todoCommandService.delete(id);
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send("delete todo successfully!");

    }
}

```

Here is the service.yml file that wire all components together. 

```yaml
# Singleton service factory configuration/IoC injection
singletons:
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      jdbcUrl: jdbc:mysql://localhost:3306/eventuate?useSSL=false
      username: mysqluser
      password: mysqlpw
      maximumPoolSize: 15
      useServerPrepStmts: true
      cachePrepStmts: true
      cacheCallableStmts: true
      prepStmtCacheSize: 4096
      prepStmtCacheSqlLimit: 2048
- com.networknt.eventuate.jdbc.IdGenerator:
  - com.networknt.eventuate.jdbc.IdGeneratorImpl
- com.networknt.tram.todolist.command.TodoCommandService:
  - com.networknt.tram.todolist.command.TodoCommandService


# OpenAPI parser validator implementation
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Callback>:
  - com.networknt.oas.validator.impl.CallbackValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Contact>:
  - com.networknt.oas.validator.impl.ContactValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.EncodingProperty>:
  - com.networknt.oas.validator.impl.EncodingPropertyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Example>:
  - com.networknt.oas.validator.impl.ExampleValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ExternalDocs>:
  - com.networknt.oas.validator.impl.ExternalDocsValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Header>:
  - com.networknt.oas.validator.impl.HeaderValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.License>:
  - com.networknt.oas.validator.impl.LicenseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Info>:
  - com.networknt.oas.validator.impl.InfoValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Link>:
  - com.networknt.oas.validator.impl.LinkValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Schema>:
  - com.networknt.oas.validator.impl.SchemaValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.MediaType>:
  - com.networknt.oas.validator.impl.MediaTypeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OAuthFlow>:
  - com.networknt.oas.validator.impl.OAuthFlowValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OpenApi3>:
  - com.networknt.oas.validator.impl.OpenApi3Validator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.RequestBody>:
  - com.networknt.oas.validator.impl.RequestBodyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Operation>:
  - com.networknt.oas.validator.impl.OperationValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Parameter>:
  - com.networknt.oas.validator.impl.ParameterValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Path>:
  - com.networknt.oas.validator.impl.PathValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Response>:
  - com.networknt.oas.validator.impl.ResponseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityRequirement>:
  - com.networknt.oas.validator.impl.SecurityRequirementValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityScheme>:
  - com.networknt.oas.validator.impl.SecuritySchemeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Server>:
  - com.networknt.oas.validator.impl.ServerValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ServerVariable>:
  - com.networknt.oas.validator.impl.ServerVariableValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Tag>:
  - com.networknt.oas.validator.impl.TagValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Xml>:
  - com.networknt.oas.validator.impl.XmlValidator
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
  - com.networknt.tram.todolist.command.PathHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
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
  # Parsing OpenAPI 3.0 specification based on request uri and method.
  - com.networknt.openapi.OpenApiHandler
  # Security JWT token verification and scope verification (depending on OpenApiHandler)
  - com.networknt.openapi.JwtVerifyHandler
  # Body Parse body based on content type in the header.
  - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on OpenAPI 3.0 specification (depending on OpenApiHandler and BodyHandler)
  - com.networknt.openapi.ValidatorHandler


``` 

In the next step, we are going to take a look at [view side restful service][].



[light-codegen tutorial]: /tutorial/generator/
[view side restful service]: /tutorial/tram/todo-list/view-service/

