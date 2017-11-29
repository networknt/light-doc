---
title: "Httpschain"
date: 2017-11-29T16:08:33-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


Now these APIs are working as a chain with Http connection. In this step, we are going to
change the connection to Https and see what is the performance difference. 

#### Prepare Environment

Before starting this step, let's create a folder called httpschain in each sub folder under
ms_chain and copy everything from httpchain folder to the httpschain. We are going to update
httpschain folder to have connections switched to Https by changing the urls in config files.

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_a
cp -r httpchain httpschain
cd ~/networknt/light-example-4j/rest/ms_chain/api_b
cp -r httpchain httpschain
cd ~/networknt/light-example-4j/rest/ms_chain/api_c
cp -r httpchain httpschain
cd ~/networknt/light-example-4j/rest/ms_chain/api_d
cp -r httpchain httpschain

```

Now we have httpschain folder copied from httpchain and all updates in this step will be
in httpschain folder. 

#### API D

As we have the server.yml to enable both http and https so the server config for
API D is ready. However, we only tried to access the API D https port through curl with
security disabled. Java client access to API through https is not an easy task so we are
going to complete the test cases to test API D TLS connection with client module first.

This step also shows how to build end-to-end test case to test your API endpoints.

Locate DataGetHandlerTest from src/test/com/networknt/apid/handler folder and you can see
there is a test case and the body is commented out. Let's uncomment the test and run it.

As the server.yml contains enableHttps so that httpsPort will be used automatically. 

```
package com.networknt.apid.handler;

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

import java.io.IOException;


public class DataGetHandlerTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(DataGetHandlerTest.class);
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;

    @Test
    public void testDataGetHandlerTest() throws ClientException, ApiException {
        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setPath("/v1/data").setMethod(Methods.GET);
            
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            
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

As the server is started with self-signed key pair so we have to make sure client has a trust
store with server certificate in order to pass the server certificate verification. These keys
should be changed when you deploy to official environment.

In our test case, we are using our own client module to access the server and a client.yml is
already included as enableClient is true in config.json for the generator. client.yml file, 
it refers to client.truststore and client.keystore

These files are part of the client module and they are added automatically by light-codegen  

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_d/httpschain
mvn clean install exec:exec
```

If above mvn command starts the server successfully, that means the test case we put in is
passed. 

#### API C
Let's leave API D running and update API C api_c.yml to use https url instead of
http. 

Locate api_c.yml in src/main/resources/config folder.

```
api_d_host: https://localhost:7444
api_d_path: /v1/data
```


Start API C server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_c/httpschain
mvn clean install exec:exec
```
From another terminal window run:

```
curl -k https://localhost:7443/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2"]
```

As we've switched to https, from this moment on, we will be using https url all the time.

#### API B

Let's keep API C and API D running and update api_b.yml in src/main/resources/config folder.

```
api_c_host: https://localhost:7443
api_c_path: /v1/data

```


Start API B server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_b/httpschain
mvn clean install exec:exec
```
From another terminal window run:

```
curl -k https://localhost:7442/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2"]
```


#### API A

API A will call API B to fulfill its request and we need to update api_a.yml in 
src/main/resources/config folder.

```
api_b_host: https://localhost:7442
api_b_path: /v1/data
```

Start API A server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_a/httpschain
mvn clean install exec:exec
```
From another terminal window run:

```
curl -k https://localhost:7441/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

At this moment, we have all four APIs completed and A is calling B, B is calling C and
C is calling D using Https connections.

