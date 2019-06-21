---
title: Multiple Instances of the same Service
linktitle: Multiple Instances of the same Service
date: 2017-10-17T19:08:51-04:00
lastmod: 2018-05-15
description: "Client-side load balancing through direct service discovery using configuration with multiple instances."
weight: 40
sections_weight: 40
draft: false
toc: true
---

## Introduction

In this step, we're going to start two `API D` instances that will listen to `7444` and `7445`.

Let's copy our current state from the last step into the `multiple` directory.

```bash
cd ~/networknt
cp -r light-example-4j/discovery/api_a/dynamic/ light-example-4j/discovery/api_a/multiple
cp -r light-example-4j/discovery/api_b/dynamic/ light-example-4j/discovery/api_b/multiple
cp -r light-example-4j/discovery/api_c/dynamic/ light-example-4j/discovery/api_c/multiple
cp -r light-example-4j/discovery/api_d/dynamic/ light-example-4j/discovery/api_d/multiple
```

## Configuring the Servers
 
### API B 

Let's modify API B service.yml to have two API D instances that listen to 7444 and 7445.

```yaml
singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          parameters:
            com.networknt.ad-1.0.0: https://localhost:7444,https://localhost:7445
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
```

Also, to see different endpoints being hit, let's disable the connection caching in the request to `API D`.

Change the following section in `DataGetHandler.java`:

```java
package com.networknt.ab.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.handler.LightHttpHandler;
import com.networknt.security.JwtHelper;
import com.networknt.server.Server;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements LightHttpHandler {
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    static String apidHost;
    static String path = "/v1/data";
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static String tag = Server.getServerConfig().getEnvironment();

    static Http2Client client = Http2Client.getInstance();

    public DataGetHandler() {
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        final CountDownLatch latch = new CountDownLatch(1);
        ClientConnection connection = null;
        try {
            apidHost = cluster.serviceToUrl("https", "com.networknt.ad-1.0.0", tag, null);
            connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exeption:", e);
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(path);
            // this is to ask client module to pass through correlationId and traceabilityId as well as
            // getting access token from oauth2 server automatically and attatch authorization headers.
            if(securityEnabled) client.propagateHeaders(request, exchange);
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            if(statusCode >= 300){
                throw new Exception("Failed to call API D: " + statusCode);
            }
            List<String> apidList = Config.getInstance().getMapper().readValue(reference.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apidList);
        } catch (Exception e) {
            logger.error("Exception:", e);
            throw new ClientException(e);
        }
        list.add("API B: Message 1");
        list.add("API B: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}

```

### API D

In order to start two instances with the same code base, we need to modify the server.yml before starting the second server. 

Also, let's update the handler so that we know which port serves the request.

`DataGetHandler.java`

```java
package com.networknt.ad.handler;

import com.networknt.config.Config;
import com.networknt.handler.LightHttpHandler;
import com.networknt.server.Server;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DataGetHandler implements LightHttpHandler {
    
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        int port = Server.getServerConfig().getHttpsPort();
        List<String> messages = new ArrayList<>();
        messages.add("API D: Message 1 from port " + port);
        messages.add("API D: Message 2 from port " + port);
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}

```

## Starting the Servers

Now let's start all five servers from five terminals. API D has two instances.

**API A**

```
cd ~/networknt/light-example-4j/discovery/api_a/multiple
mvn clean install -Prelease
java -jar target/aa-1.0.0.jar
```

**API B**

```
cd ~/networknt/light-example-4j/discovery/api_b/multiple
mvn clean install -Prelease
java -jar target/ab-1.0.0.jar
```

**API C**

```
cd ~/networknt/light-example-4j/discovery/api_c/multiple
mvn clean install -Prelease
java -jar target/ac-1.0.0.jar
```

**API D**


And start the first instance that listen to 7444.

```
cd ~/networknt/light-example-4j/discovery/api_d/multiple
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```
 
Now let's start the second instance of `API D`. Before starting the server, let's update server.yml with https port 7445.

```yaml
httpsPort: 7445
```

And start the second instance

```
cd ~/networknt/light-example-4j/discovery/api_d/multiple
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

## Testing the Servers

```
curl -k https://localhost:7441/v1/data
```

And the result can be from port 7444 or 7445 as Round Robin load balancer will pick up on of them.

```
["Message 1 from port 7444","Message 2 from port 7444","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

Or

```
["Message 1 from port 7445","Message 2 from port 7445","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

The next step, we are going to use [Consul][] to do the service registry and discovery.

[Consul]: /tutorial/common/discovery/consul/