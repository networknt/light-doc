---
title: "Test Service1"
date: 2019-04-04T10:25:55-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [generated service1][] project. In this step, we are going to work with the service1 project in the IntelliJ IDEA. 

If you don't have IntelliJ community edition installed, please [install][] it, and it is free. You can use Eclipse as well if it is your preferred IDE. 

We assume that you are using IntelliJ IDEA for the following steps. 

Open the service1 project from light-example-4j/hybrid/hello-world/service1

In the src/main, there are two handlers in the `com.networknt.hello.handler` package. 

Info.java

```

package com.networknt.hello.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/service1/info/0.1.0")
public class Info implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("");
    }
}
```

Let's change the handler method a little bit to return "info" instead of "". 

After the change, the file should be. 

```

package com.networknt.hello.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/service1/info/0.1.0")
public class Info implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("info");
    }
}

```

Let's change the Query.java to return "query". 

Query.java

```
package com.networknt.hello.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/service1/query/0.1.0")
public class Query implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("query");
    }
}
```

Here we just want to return a string for the simple test. In a real service, you might have complicated logic and return a JSON instead. 

With the two handlers updated, let's update the test cases in the src/test folder in `com.networknt.hello.handler` package. 

The generated InfoTest.java should look like. 

```

package com.networknt.hello.handler;

import com.networknt.client.Http2Client;
import com.networknt.exception.ApiException;
import com.networknt.exception.ClientException;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.junit.Assert;
import org.junit.ClassRule;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.IoUtils;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class InfoTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(Info.class); 
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;

    @Test
    public void testInfo() throws ClientException, ApiException {
        /*
        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setPath("/api/json").setMethod(Methods.POST);
            request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
            request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
            connection.sendRequest(request, client.createClientCallback(reference, latch, "request body to be replaced"));
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            IoUtils.safeClose(connection);
        }
        int statusCode = reference.get().getResponseCode();
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Assert.assertEquals(200, statusCode);
        Assert.assertNotNull(body);
        */
    }
}

```

As you can see, the body of the test case is commented out. This is due to we don't know the request body and response body assertion in the light-codegen. 

Let's uncomment the code and replace the request body with the following. 

```
{"host":"lightapi.net","service":"service1","action":"info","version":"0.1.0","data":{"filter":"value1"}}
```

The updated InfoTest.java should look like this.

```
package com.networknt.hello.handler;

import com.networknt.client.Http2Client;
import com.networknt.exception.ApiException;
import com.networknt.exception.ClientException;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.junit.Assert;
import org.junit.ClassRule;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.IoUtils;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class InfoTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(Info.class); 
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;

    @Test
    public void testInfo() throws ClientException, ApiException {

        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setPath("/api/json").setMethod(Methods.POST);
            request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
            request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
            connection.sendRequest(request, client.createClientCallback(reference, latch, "{\"host\":\"lightapi.net\",\"service\":\"service1\",\"action\":\"info\",\"version\":\"0.1.0\",\"data\":{\"filter\":\"value1\"}}"));
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            IoUtils.safeClose(connection);
        }
        int statusCode = reference.get().getResponseCode();
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Assert.assertEquals(200, statusCode);
        Assert.assertNotNull(body);

    }
}

```

Now the test case should be passed if you run it from the IDE. You can write negative test cases if you want. For example, change the data properties to move the filter which is required by the schema on the server. 

let's change the QueryTest.java to the following.

```
package com.networknt.hello.handler;

import com.networknt.client.Http2Client;
import com.networknt.exception.ApiException;
import com.networknt.exception.ClientException;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.junit.Assert;
import org.junit.ClassRule;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.IoUtils;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class QueryTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(Query.class); 
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;

    @Test
    public void testQuery() throws ClientException, ApiException {
        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setPath("/api/json").setMethod(Methods.POST);
            request.getRequestHeaders().put(Headers.CONTENT_TYPE, "application/json");
            request.getRequestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
            connection.sendRequest(request, client.createClientCallback(reference, latch, "{\"host\":\"lightapi.net\",\"service\":\"service1\",\"action\":\"query\",\"version\":\"0.1.0\",\"data\":{\"param1\":\"value1\",\"param2\":\"value2\"}}"));
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            IoUtils.safeClose(connection);
        }
        int statusCode = reference.get().getResponseCode();
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Assert.assertEquals(200, statusCode);
        Assert.assertNotNull(body);

    }
}

```

If you run the test case, it should pass. If you remove the param2 from the body, you will receive an error as the param1 and param2 are mandatory in the schema. In light-hybrid-4j, we are loading the schema during the runtime to perform the validation for the request. 

At this stage, we are sure that the service1 is working as expected. You can write more test cases to cover different combinations for service1. 

In the next step, we are going to build servie1 and [deploy][] it to the server instance. 

[generated service1]: /tutorial/hybrid/hello-world/generate-service1/
[install]: /tool/idea/
[deploy]: /tutorial/hybrid/hello-world/deploy-service1/
