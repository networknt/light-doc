---
date: 2017-10-17T18:46:16-04:00
title: Dynamic service discovery with direct registry
---

Previous step: [Static][]

In dynamic discovery, we are going to use cluster components with direct registry
so that we don't need to start external consul or zookeeper instances. We still go 
through registry for service discovery but the registry is defined in service.yml.

With direct registry in service.yml, we can easily move to consul for service
discovery without changing the code. Each service can be package with docker
and utilize different service discovery option by change the configuration in
service.yml file.


First let's copy our previous static code into the to dynamic folder.

```
cp -r ~/networknt/discovery/api_a/static/ ~/networknt/discovery/api_a/dynamic
cp -r ~/networknt/discovery/api_b/static/ ~/networknt/discovery/api_b/dynamic
cp -r ~/networknt/discovery/api_c/static/ ~/networknt/discovery/api_c/dynamic
cp -r ~/networknt/discovery/api_d/static/ ~/networknt/discovery/api_d/dynamic
```

Lets begin by updating our `API A` service to dynamically resolve the host and
ports of `API B` and `API C` based on given service ids.

### API A

`DataGetHandler.java`

```java
package com.networknt.apia.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.server.Server;
import com.networknt.service.SingletonServiceFactory;
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
    private static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    private static String tag = Server.config.getEnvironment();

    private static Http2Client client = Http2Client.getInstance();
    private static ClientConnection connectionB;
    private static ClientConnection connectionC;

    private ClientConnection getConnectionB() throws Exception {
        String apibHost = cluster.serviceToUrl("https", "com.networknt.apib-1.0.0", tag, null);
        return client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
    }

    private ClientConnection getConnectionC() throws Exception {
        String apicHost = cluster.serviceToUrl("https", "com.networknt.apic-1.0.0", tag, null);
        return client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (connectionB == null || !connectionB.isOpen()) {
            connectionB = getConnectionB();
        }
        if (connectionC == null || !connectionC.isOpen()) {
            connectionC = getConnectionC();
        }

        final CountDownLatch latch = new CountDownLatch(2);
        final AtomicReference<ClientResponse> referenceB = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceC = new AtomicReference<>();

        String requestPath = "/v1/data";
        ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(requestPath);
        connectionB.sendRequest(requestB, client.createClientCallback(referenceB, latch));

        ClientRequest requestC = new ClientRequest().setMethod(Methods.GET).setPath(requestPath);
        connectionC.sendRequest(requestC, client.createClientCallback(referenceC, latch));

        latch.await();

        int statusCodeB = referenceB.get().getResponseCode();
        if(statusCodeB >= 300) throw new Exception("Failed to call API B: " + statusCodeB);
        List<String> apibList = Config.getInstance().getMapper().readValue(referenceB.get().getAttachment(Http2Client.RESPONSE_BODY), new TypeReference<List<String>>(){});

        List<String> results = new ArrayList<>(apibList);

        int statusCodeC = referenceC.get().getResponseCode();
        if(statusCodeC >= 300) throw new Exception("Failed to call API C: " + statusCodeC);
        List<String> apicList = Config.getInstance().getMapper().readValue(referenceC.get().getAttachment(Http2Client.RESPONSE_BODY), new TypeReference<List<String>>(){});
        results.addAll(apicList);

        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(results));
    }
}
```

The service configuration also needs to be updated to inject the singleton implementations of
`Cluster`, `LoadBanlance`, `URL` and `Registry`. Please note that the key in parameters
is serviceId of your calling APIs

The following should be added to the `service.yml` `singletons` list:

```yaml
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      parameters:
        com.networknt.apib-1.0.0: https://localhost:7442
        com.networknt.apic-1.0.0: https://localhost:7443
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

As we don't need `api_a.yml` anymore, it can be removed.

Now we do the same for `API B` which needs to resolve the endpoint of 
`API D`. 

### API B

`DataGetHandler.java`

```java
package com.networknt.apib.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.server.Server;
import com.networknt.service.SingletonServiceFactory;
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
    private static Http2Client client = Http2Client.getInstance();
    private static ClientConnection connection;

    private static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    private static String tag = Server.config.getEnvironment();

    private ClientConnection getConnectionD() throws Exception {
        String apidHost = cluster.serviceToUrl("https", "com.networknt.apid-1.0.0", tag, null);
        return client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        final CountDownLatch latch = new CountDownLatch(1);
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        if(connection == null || !connection.isOpen()) {
            connection = getConnectionD();
        }

        ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath("/v1/data");
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

The following should be added to the `service.yml` `singletons` list:

```yaml
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      parameters:
        com.networknt.apid-1.0.0: https://localhost:7444
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

### Start Servers

Now let's start all four servers from four terminals.

**API A**

```
cd ~/networknt/discovery/api_a/dynamic
mvn clean install exec:exec
```

**API B**

```
cd ~/networknt/discovery/api_b/dynamic
mvn clean install exec:exec

```

**API C**

```
cd ~/networknt/discovery/api_c/dynamic
mvn clean install exec:exec

```

**API D**

```
cd ~/networknt/discovery/api_d/dynamic
mvn clean install exec:exec

```

### Test Servers

Let's access API A and see if we can get messages from all four servers.

```
curl -k https://localhost:7441/v1/data

```
The result is 

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2"]
```

In the next step we are going to start multiple instances of `API D` and see 
how direct registry will handle the situation.

Next Step: [Multiple]({{< relref "/tutorial/common/discovery/multiple.md" >}})

[Static]: /tutorial/common/discovery/static/
[consul]: /tutorial/common/discovery/consul/
[multiple]: /tutorial/common/discovery/multiple/
