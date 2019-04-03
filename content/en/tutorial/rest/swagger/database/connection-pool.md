---
title: "Connection Pool"
date: 2017-11-24T11:41:13-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 70      #rem
aliases: []
toc: false
draft: false
reviewed: true
---

To connect to the database we need to have service.yml that can inject connection pool to the microservice you are building.

Now we have a generated project with configuration for MySQL, let's take a look at the generated service.yml, and you can see the following section for MySQL connection pool configuration. 

service.yml in ~/networknt/light-example-4j/rest/swagger/database/generated/src/main/resources/config

```
# Singleton service factory configuration/IoC injection
singletons:
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.database.PathHandlerProvider
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


- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://mysqldb:3306/oauth2?useSSL=false
      username: root
      password: my-secret-pw
      maximumPoolSize: 10
      useServerPrepStmts: true,
      cachePrepStmts: true,
      cacheCallableStmts: true,
      prepStmtCacheSize: 10,
      prepStmtCacheSqlLimit: 2048,
      connectionTimeout: 2000

```

As you can see, the last section is the DataSource configuration for MySQL database. 

Before we make any change, let's create a new folder so that we can always compare with the generated folder if needed. 
 

```
cd ~/networknt/light-example-4j/rest/swagger/database
cp -r generated connection
```

The service.yaml will make sure that a Hikari DataSource will be created during server startup with the dependency injection module. You can find other database's service.yml in 
https://github.com/networknt/light-example-4j/tree/master/rest/swagger/database/dbscript

As we are running the service as a standalone Java application without Docker, we need to change the service.yml for the jdbcUrl to point the database to localhost instead of mysqldb. Also, we need to switch our database to hello_world from oauth2. 

Here is the updated DataSource.


```
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://localhost:3306/hello_world?useSSL=false
      username: root
      password: my-secret-pw
      maximumPoolSize: 10
      useServerPrepStmts: true,
      cachePrepStmts: true,
      cacheCallableStmts: true,
      prepStmtCacheSize: 10,
      prepStmtCacheSqlLimit: 2048,
      connectionTimeout: 2000
```

In the config.json for light-codegen, we've enabled MySQL databases support so that pom.xml should have all the dependencies included. 

Now you can add a line in each handler to get the DataSource as a static variable.


```
    private static final DataSource ds = SingletonServiceFactory.getBean(DataSource.class);

```

If you are using IDE, it will help you to add imports, otherwise, you have to add the following imports.

```
import com.networknt.service.SingletonServiceFactory;
import javax.sql.DataSource;

```

If you can build, start and access the server with curl, that means the database connection is created although we are not using it in our code yet. The next step we will try to [query database][].

[query database]: /tutorial/rest/swagger/database/single-query/