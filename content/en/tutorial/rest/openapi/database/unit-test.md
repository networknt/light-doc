---
title: "Unit Test"
date: 2017-11-24T17:32:45-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 100     #rem
aliases: []
toc: false
draft: false
reviewed: true
---

In previous steps, we have updated three endpoints in the service and switch from Mysql to Oracle and Postgres without changing the code. In an official environment, config files like service.yml will be externalized so that you can even switch database during runtime just restart your instance whether your service is running as a Java process with "java -jar" or in Docker container.

In this step, we are going to create some unit test cases. As these tests are very important in ensuring the server you build works. They give us confidence in changing the code and support continuous integration to production.
   
Given our code is based on a lightweight Http framework, all our tests will be using the real server. There is a TestServer.java in the generated code already, and each handler will have a generated test case. The only thing we need to do is to add testing logic.

First, let's create a new folder call test from the postgres folder which is using Postgres database.

```
cd ~/networknt/light-example-4j/rest/openapi/database
cp -r postgres test
```

Now let's go to IDE and navigate to the test folder under src. You can find there are three test cases for each handler and there is an extra class called TestServer.java

Let's take a look at the generated QueryGetHandlerTest.java

```java
package com.networknt.database.handler;

import com.networknt.client.Http2Client;
import com.networknt.exception.ClientException;
import com.networknt.openapi.ResponseValidator;
import com.networknt.schema.SchemaValidatorsConfig;
import com.networknt.status.Status;
import com.networknt.utility.StringUtils;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.HeaderValues;
import io.undertow.util.HttpString;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.junit.Assert;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.Ignore;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.IoUtils;
import org.xnio.OptionMap;
import java.net.URI;
import java.util.Optional;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;


@Ignore
public class QueryGetHandlerTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(QueryGetHandlerTest.class);
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;
    static final String JSON_MEDIA_TYPE = "application/json";

    @Test
    public void testQueryGetHandlerTest() throws ClientException {

        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        String requestUri = "/v1/query";
        String httpMethod = "get";
        try {
            ClientRequest request = new ClientRequest().setPath(requestUri).setMethod(Methods.GET);
            
            //customized header parameters 
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            IoUtils.safeClose(connection);
        }
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Optional<HeaderValues> contentTypeName = Optional.ofNullable(reference.get().getResponseHeaders().get(Headers.CONTENT_TYPE));
        SchemaValidatorsConfig config = new SchemaValidatorsConfig();
        config.setMissingNodeAsError(true);
        ResponseValidator responseValidator = new ResponseValidator(config);
        int statusCode = reference.get().getResponseCode();
        Status status;
        if(contentTypeName.isPresent()) {
            status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), contentTypeName.get().getFirst());
        } else {
            status = responseValidator.validateResponseContent(body, requestUri, httpMethod, String.valueOf(statusCode), JSON_MEDIA_TYPE);
        }
        Assert.assertNull(status);
    }
}

```

In this test file, we can see that a static TestServer instance is started with @ClassRule to be shared by all test cases in this test file. This is a positive test generated based on OpenAPI 3.0 specification. The result checking code is validation against the schema in the specification because we don't know what is the exact object returned. You can modify the test case to assert the response body or status code to ensure that the response is what you are expecting. 

Let's modify this test to make it work in our service and connect to the H2 database for local testing without an external database. The H2 dependency has been added in light-codegen config.json

Note that if we run the test case above, we are still using the Postgres database which is configured in src/main/resources/config/service.yml file. It requires an external database running locally. In order to remove this dependency, let's use the H2 embedded database for the unit test.   

First, we need to prepare the H2 database script. Here is the create_h2.sql file that needs to be created in src/test/resources. You can copy the existing file from database.bak folder.

```sql
DROP table IF EXISTS world;

CREATE TABLE  world (
  id int auto_increment primary key,
  randomNumber int NOT NULL
);


INSERT INTO world(id, randomNumber) VALUES (1, 10);
INSERT INTO world(id, randomNumber) VALUES (2, 9);
INSERT INTO world(id, randomNumber) VALUES (3, 8);
INSERT INTO world(id, randomNumber) VALUES (4, 7);
INSERT INTO world(id, randomNumber) VALUES (5, 6);
INSERT INTO world(id, randomNumber) VALUES (6, 5);
INSERT INTO world(id, randomNumber) VALUES (7, 4);
INSERT INTO world(id, randomNumber) VALUES (8, 3);
INSERT INTO world(id, randomNumber) VALUES (9, 2);
INSERT INTO world(id, randomNumber) VALUES (10, 1);

```

And then we need to change TestServer.java to initialize H2 database. Here is the updated file.

```java

package com.networknt.database.handler;

import com.networknt.server.Server;
import com.networknt.service.SingletonServiceFactory;
import org.h2.tools.RunScript;
import org.junit.rules.ExternalResource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.InputStream;
import java.io.InputStreamReader;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.concurrent.atomic.AtomicInteger;

import com.networknt.server.Server;
import com.networknt.server.ServerConfig;

import javax.sql.DataSource;

import static java.nio.charset.StandardCharsets.UTF_8;

public class TestServer extends ExternalResource {
    static final Logger logger = LoggerFactory.getLogger(TestServer.class);

    private static final AtomicInteger refCount = new AtomicInteger(0);
    private static Server server;

    private static final TestServer instance  = new TestServer();

    private TestServer() {
        DataSource ds = (DataSource) SingletonServiceFactory.getBean(DataSource.class);
        try (Connection connection = ds.getConnection()) {
            String schemaResourceName = "/create_h2.sql";
            InputStream in = TestServer.class.getResourceAsStream(schemaResourceName);

            if (in == null) {
                throw new RuntimeException("Failed to load resource: " + schemaResourceName);
            }
            InputStreamReader reader = new InputStreamReader(in, UTF_8);
            RunScript.execute(connection, reader);

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static TestServer getInstance () {
        return instance;
    }


    public ServerConfig getServerConfig() {
        return Server.config;
    }

    @Override
    protected void before() {
        try {
            if (refCount.get() == 0) {
                Server.start();
            }
        }
        finally {
            refCount.getAndIncrement();
        }
    }

    @Override
    protected void after() {
        refCount.getAndDecrement();
        if (refCount.get() == 0) {
            Server.stop();
        }
    }
}

```

Next, we need to create another service.yml in src/test/resources/config folder to overwrite the one in src/main/resources/config folder which is pointing to Postgres.

Here is the one for testing with the H2 data source.

```
# Singleton service factory configuration
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: org.h2.jdbcx.JdbcDataSource
      jdbcUrl: jdbc:h2:~/test
      username: sa
      password: sa
      maximumPoolSize: 10
      useServerPrepStmts: true,
      cachePrepStmts: true,
      cacheCallableStmts: true,
      prepStmtCacheSize: 10,
      prepStmtCacheSqlLimit: 2048,
      connectionTimeout: 2000

```

Now you can run the test cases, and it should be passed with the H2 database as backend. 

In the [next step][], we are going to do some performance test with the Postgres database. 

[next step]: /tutorial/rest/openapi/database/performance/

