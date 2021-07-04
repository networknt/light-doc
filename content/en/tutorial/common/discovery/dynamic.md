---
title: Dynamic Service Discovery with Direct Registry
linktitle: Dynamic Service Discovery with Direct Registry
date: 2017-10-17T18:46:16-04:00
lastmod: 2018-05-15
description: "Service discovery through endpoints defined in a configuration allows a seamless move to external registries without code changes."
weight: 30
sections_weight: 30
draft: false
toc: true
reviewed: true
---

## Introduction

In dynamic discovery, we are going to use the cluster components with the direct registry so that we don't need to start external Consul or Zookeeper instances. We still go through the registry for service discovery, but the registry is defined in service.yml.

With direct registry in service.yml, we can easily move to [Consul] for service discovery without changing the code. Each service can be packaged with Docker and utilize different service discovery option by changing the configuration in the service.yml file.

First, let's copy our previous static code into the dynamic folder.

```
cd ~/networknt
cp -r light-example-4j/discovery/api_a/static/ light-example-4j/discovery/api_a/dynamic
cp -r light-example-4j/discovery/api_b/static/ light-example-4j/discovery/api_b/dynamic
cp -r light-example-4j/discovery/api_c/static/ light-example-4j/discovery/api_c/dynamic
cp -r light-example-4j/discovery/api_d/static/ light-example-4j/discovery/api_d/dynamic
```

Let's begin by updating our `API A` service to dynamically resolve the host and ports of `API B` and `API C` based on given service ids.

## Configuring the APIs

### API A

`DataGetHandler.java`

```java
package com.networknt.aa.handler;

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
    static String apibHost;
    static String apicHost;
    static String path = "/v1/data";
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static String tag = Server.getServerConfig().getEnvironment();

    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connectionB;
    static ClientConnection connectionC;

    public DataGetHandler() {
        try {
            apibHost = cluster.serviceToUrl("https", "com.networknt.ab-1.0.0", tag, null);
            apicHost = cluster.serviceToUrl("https", "com.networknt.ac-1.0.0", tag, null);
            connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exeption:", e);
        }
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        if(connectionB == null || !connectionB.isOpen()) {
            try {
                apibHost = cluster.serviceToUrl("https", "com.networknt.ab-1.0.0", tag, null);
                connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        if(connectionC == null || !connectionC.isOpen()) {
            try {
                apicHost = cluster.serviceToUrl("https", "com.networknt.ac-1.0.0", tag, null);
                connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final CountDownLatch latch = new CountDownLatch(2);
        final AtomicReference<ClientResponse> referenceB = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceC = new AtomicReference<>();
        try {
            ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestB, exchange);
            connectionB.sendRequest(requestB, client.createClientCallback(referenceB, latch));

            ClientRequest requestC = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestC, exchange);
            connectionC.sendRequest(requestB, client.createClientCallback(referenceC, latch));

            latch.await();

            int statusCodeB = referenceB.get().getResponseCode();
            if(statusCodeB >= 300){
                throw new Exception("Failed to call API B: " + statusCodeB);
            }
            List<String> apibList = Config.getInstance().getMapper().readValue(referenceB.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apibList);

            int statusCodeC = referenceC.get().getResponseCode();
            if(statusCodeC >= 300){
                throw new Exception("Failed to call API C: " + statusCodeC);
            }
            List<String> apicList = Config.getInstance().getMapper().readValue(referenceC.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apicList);
        } catch (Exception e) {
            logger.error("Exception:", e);
            throw new ClientException(e);
        }
        list.add("API A: Message 1");
        list.add("API A: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}
```

The service configuration also needs to be updated to inject the singleton implementations of `Cluster`, `LoadBalance`, `URL` and `Registry`. Please note that the key in parameters is serviceId of your calling APIs

The following should be added to the `service.yml` `singletons` list:

```yaml
singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          parameters:
            com.networknt.ab-1.0.0: https://localhost:7442
            com.networknt.ac-1.0.0: https://localhost:7443
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
```

As we don't need `api_a.yml` anymore, it can be removed.

Now we do the same for `API B` which needs to resolve the endpoint of `API D`. 

### API B

`DataGetHandler.java`

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
    static ClientConnection connection;

    public DataGetHandler() {
        try {
            apidHost = cluster.serviceToUrl("https", "com.networknt.ad-1.0.0", tag, null);
            connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exeption:", e);
        }
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        final CountDownLatch latch = new CountDownLatch(1);
        if(connection == null || !connection.isOpen()) {
            try {
                apidHost = cluster.serviceToUrl("https", "com.networknt.ad-1.0.0", tag, null);
                connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
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

The following should be added to the `service.yml` `singletons` list:

```yaml
singletons:
  - com.networknt.registry.URL:
      - com.networknt.registry.URLImpl:
          parameters:
            com.networknt.ad-1.0.0: https://localhost:7444
  - com.networknt.registry.Registry:
      - com.networknt.registry.support.DirectRegistry
  - com.networknt.balance.LoadBalance:
      - com.networknt.balance.RoundRobinLoadBalance
  - com.networknt.cluster.Cluster:
      - com.networknt.cluster.LightCluster
```

As we don't need `api_b.yml` anymore, it can be removed.

### API C

API C is not calling any other APIs, so there is no change to its handler.

### API D

API D is not calling any other APIs, so there is no change to its handler.

## Starting the Servers

Now let's start all four servers from four terminals.

**API A**

```
cd ~/networknt/light-example-4j/discovery/api_a/dynamic
mvn clean install -Prelease
java -jar target/aa-1.0.0.jar
```

**API B**

```
cd ~/networknt/light-example-4j/discovery/api_b/dynamic
mvn clean install -Prelease
java -jar target/ab-1.0.0.jar
```

**API C**

```
cd ~/networknt/light-example-4j/discovery/api_c/dynamic
mvn clean install -Prelease
java -jar target/ac-1.0.0.jar
```

**API D**

```
cd ~/networknt/light-example-4j/discovery/api_d/dynamic
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

## Testing the Servers

Let's access API A and see if we can get messages from all four servers.

```
curl -k https://localhost:7441/v1/data

```
The result is 

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

In the next step, we are going to start [multiple][] instances of `API D` and see how direct registry will handle the situation.

[Consul]: /tutorial/common/discovery/consul/
[multiple]: /tutorial/common/discovery/multiple/
