---
title: Static Configuration
linktitle: Static Configuration
date: 2017-10-17T18:12:46-04:00
lastmod: 2018-05-15
description: "Statically connecting microservices in a realistic call hierarchy where each
service is hard-bounded to a known ip and host."
weight: 20
sections_weight: 20
draft: false
toc: true
reviewed: true
---

## Introduction

Now that we have four standalone services, the next step is to connect them together.

Here is the call tree for these services:

```
API A -> API B -> API D
      -> API C
```

Before we change the code, let's copy the generated projects to new folders so that we can compare the changes later on.

```
cp -r ~/networknt/discovery/api_a/generated/ ~/networknt/discovery/api_a/static
cp -r ~/networknt/discovery/api_b/generated/ ~/networknt/discovery/api_b/static
cp -r ~/networknt/discovery/api_c/generated/ ~/networknt/discovery/api_c/static
cp -r ~/networknt/discovery/api_d/generated/ ~/networknt/discovery/api_d/static
```

Let's start updating the code in static folders for each project. If you are using IntelliJ IDEA Community Edition, you need to open the discovery folder in `~/networknt` and then import each project by right click pom.xml from the static folder and select Add as Maven project.

As indicated from the title, we're going to hard-code URLs in API to API calls within configuration files (i.e. services will be deployed on the known hosts with known ports)

## Configuring the APIs 

### API A

For `API A`, as it is calling `API B` and `API C`, its handler needs to be changed to call two other APIs and needs to load a configuration file that defines the URLs for `API B` and `API C`.

`DataGetHandler.java`

```java
package com.networknt.apia.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    private static String CONFIG_NAME = "api_a";

    private static String apibHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_b_host");
    private static String apibPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_b_path");
    private static String apicHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_host");
    private static String apicPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_path");

    private static Http2Client client = Http2Client.getInstance();
    private static ClientConnection connectionB;
    private static ClientConnection connectionC;

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (connectionB == null || !connectionB.isOpen()) {
            connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        }
        if (connectionC == null || !connectionC.isOpen()) {
            connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        }

        final CountDownLatch latch = new CountDownLatch(2);
        final AtomicReference<ClientResponse> referenceB = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceC = new AtomicReference<>();

        ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(apibPath);
        connectionB.sendRequest(requestB, client.createClientCallback(referenceB, latch));

        ClientRequest requestC = new ClientRequest().setMethod(Methods.GET).setPath(apicPath);
        connectionC.sendRequest(requestC, client.createClientCallback(referenceC, latch));

        latch.await();

        List<String> results = new ArrayList<>();

        int statusCodeB = referenceB.get().getResponseCode();
        if(statusCodeB >= 300) throw new Exception("Failed to call API B: " + statusCodeB);
        List<String> apibList = Config.getInstance().getMapper().readValue(referenceB.get().getAttachment(Http2Client.RESPONSE_BODY), new TypeReference<List<String>>(){});

        results.addAll(apibList);

        int statusCodeC = referenceC.get().getResponseCode();
        if(statusCodeC >= 300) throw new Exception("Failed to call API C: " + statusCodeC);
        List<String> apicList = Config.getInstance().getMapper().readValue(referenceC.get().getAttachment(Http2Client.RESPONSE_BODY), new TypeReference<List<String>>(){});
        results.addAll(apicList);

        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(results));
    }
}
```

The following is the config file that defines the URL for `API B` and `API C`. This is hardcoded and can only be changed in this config file and restart the server. For now, I am just creating the file in `src/main/resources/config` folder, but it should be externalized on the official testing environment.

`api_a.yml`

```
api_b_host: https://localhost:7442
api_b_path: /v1/data
api_c_host: https://localhost:7443
api_c_path: /v1/data
```

### API B

Change the handler to call `API D` and load configuration for `API D` URL.

`DataGetHandler.java`

```java
package com.networknt.apib.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    private static String CONFIG_NAME = "api_b";
    private static String apidHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_d_host");
    private static String apidPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_d_path");

    private static Http2Client client = Http2Client.getInstance();
    private static ClientConnection connection;

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        final CountDownLatch latch = new CountDownLatch(1);
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        if(connection == null || !connection.isOpen()) {
            connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        }

        ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(apidPath);
        connection.sendRequest(request, client.createClientCallback(reference, latch));
        latch.await();

        int statusCode = reference.get().getResponseCode();
        if(statusCode >= 300) throw new Exception("Failed to call API D: " + statusCode);

        List<String> apidList = Config.getInstance().getMapper().readValue(reference.get().getAttachment(Http2Client.RESPONSE_BODY), new TypeReference<List<String>>(){});
        List<String> results = new ArrayList<>(apidList);

        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(results));
    }
}
```

Configuration file for `API D` url:

`api_b.yml`

```
api_d_host: https://localhost:7444
api_d_path: /v1/data
```

### API C


Updated `API C` handler:

`DataGetHandler.java`

```java
package com.networknt.apic.handler;

import com.networknt.config.Config;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;

import java.util.ArrayList;
import java.util.List;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> messages = new ArrayList<>();
        messages.add("API C: Message 1");
        messages.add("API C: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}
```


### API D

Update Handler for `API D`:

`DataGetHandler.java`

```java
package com.networknt.apid.handler;

import com.networknt.config.Config;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;

import java.util.ArrayList;
import java.util.List;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> messages = new ArrayList<>();
        messages.add("API D: Message 1");
        messages.add("API D: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}
```


## Starting the Servers

Now let's start all four servers from four terminals.

**API A**

```
cd ~/networknt/discovery/api_a/static
mvn clean install exec:exec
```

**API B**

```
cd ~/networknt/discovery/api_b/static
mvn clean install exec:exec

```

**API C**

```
cd ~/networknt/discovery/api_c/static
mvn clean install exec:exec

```

**API D**

```
cd ~/networknt/discovery/api_d/static
mvn clean install exec:exec

```

## Testing the Servers

Let's access API A and see if we can get messages from all four servers.

```
curl -k https://localhost:7441/v1/data

```
The result is 

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2"]
```

For now, we have four APIs updated and with configured host and path, `API A` can call `API B`, `API C` and `API B` can call `API D`. As you can see we are using https connection between API calls.  

In the next step, we are going to switch our implementation to dynamic discovery with service.yml

Next Step: [Dynamic][]

[Dynamic]: /tutorial/common/discovery/dynamic/
