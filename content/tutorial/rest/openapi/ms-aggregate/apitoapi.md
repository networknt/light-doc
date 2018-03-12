---
title: "API to API communication"
date: 2018-03-11T20:37:18-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we have generated four APIs and tested the first one. Now these 
APIs are working if you start them and they will output the mock responses generated 
based on the API specifications. Let's take a look at the API handler itself and update 
it based on our business logic, you can call API AA and subsequently all other APIs 
will be called by API AA.

### Prepare Environment

Before starting this step, let's create a folder called https in each sub folder under
ms-aggregate and copy everything from generated folder to the https. We are going to 
update https folder to have business logic to call other APIs and change the configuration
to listen to different port. You can compare between generated and https to see what has
been changed later on.

```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/aa
cp -r generated https
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ab
cp -r generated https
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ac
cp -r generated https
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ad
cp -r generated https
```

Now we have https folder copied from generated and all updates in this step will be in 
https folder. 

### API AD
Let's take a look at the PathHandlerProvider.java in ms-aggregate/ad/https/src/main/java/com/networknt/ad

```java
package com.networknt.ad;

import com.networknt.config.Config;
import com.networknt.server.HandlerProvider;
import io.undertow.Handlers;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import com.networknt.info.ServerInfoGetHandler;
import com.networknt.health.HealthGetHandler;

import com.networknt.ad.handler.*;

public class PathHandlerProvider implements HandlerProvider {
    @Override
    public HttpHandler getHandler() {
        return Handlers.routing()
        
            .add(Methods.GET, "/v1/health", new HealthGetHandler())
        
            .add(Methods.GET, "/v1/server/info", new ServerInfoGetHandler())
        
            .add(Methods.GET, "/v1/data", new DataGetHandler())
        
        ;
    }
}
```

This is the only class that routes each endpoint defined in specification to a handler 
instance. Because we only have one endpoint /v1/data@get there is only one route added
to the handler chain. And there is a handler generated in the handler subfolder to
handle request that has the url matched to this endpoint. The /server/info is injected 
to output the server runtime information on all the components and configurations. It
will be included in every API/service. /health is another injected endpoint to output
server health check info with 200 response code and "OK" as the body. It should be
disabled in most of the case, but Kubernetes needs it. 

The generated handler is named "DataGetHandler" and it returns example response defined
in OpenAPI 3.0 specification. Here is the generated handler code. 
 
```java
package com.networknt.ad.handler;

import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
             exchange.getResponseSender().send("[\"Message 1\",\"Message 2\"]");
        
    }
}
```

Let's update it to an array of strings that indicates the response comes from API AD. 


```
package com.networknt.ad.handler;

import com.networknt.config.Config;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> messages = new ArrayList<String>();
        messages.add("API AD: Message 1");
        messages.add("API AD: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}
```

Now, let's build it and start the server. 
```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ad/https
mvn clean install exec:exec
```

Test it with curl.

```
curl -k https://localhost:7444/v1/data
```
And the result is

```
["API AD: Message 1","API AD: Message 2"]
```

Note that we have -k option to in the https command line as we are using self-signed
certificate and we don't want to verify the domain.


### API C

Let's leave API AD running and update API AC DataGetHandler in 
~/networknt/light-example-4j/rest/openapi/ms-aggregate/ac/https


```
package com.networknt.ac.handler;

import com.networknt.config.Config;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> messages = new ArrayList<String>();
        messages.add("API AC: Message 1");
        messages.add("API AC: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}

```

Start API AC server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ac/https
mvn clean install exec:exec
```
From another terminal window run:

```
curl -k https://localhost:7443/v1/data

```

And the result should be:

```
["API AC: Message 1","API AC: Message 2"]
```


### API B

Let's keep API AC and API AD running. The next step is to complete API AB. 

Now let's update the generated DataGetHandler.java to this.

```
package com.networknt.ab.handler;

import com.networknt.config.Config;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> messages = new ArrayList<String>();
        messages.add("API AB: Message 1");
        messages.add("API AB: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(messages));
    }
}
```

Start API B server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/ab/https
mvn clean install exec:exec
```

From another terminal window run:

```
curl -k https://localhost:7442/v1/data
```

And the result is:

```
["API AB: Message 1","API AB: Message 2"]
```

### API A

API AA will call API AB, AC, AD to fulfill its request. Now let's update the 
generated DataGetHandler.java code to

```
package com.networknt.aa.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.*;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static Cluster cluster = SingletonServiceFactory.getBean(Cluster.class);
    static String abHost;
    static String acHost;
    static String adHost;
    static String path = "/v1/data";
    static Map<String, Object> securityConfig = (Map) Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);

    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connectionAb;
    static ClientConnection connectionAc;
    static ClientConnection connectionAd;

    public DataGetHandler() {
        try {
            abHost = cluster.serviceToUrl("https", "com.networknt.ab-1.0.0", null, null);
            acHost = cluster.serviceToUrl("https", "com.networknt.ac-1.0.0", null, null);
            adHost = cluster.serviceToUrl("https", "com.networknt.ad-1.0.0", null, null);
            connectionAb = client.connect(new URI(abHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            connectionAc = client.connect(new URI(acHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            connectionAd = client.connect(new URI(adHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exeption:", e);
        }
    }


    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        if(connectionAb == null || !connectionAb.isOpen()) {
            try {
                abHost = cluster.serviceToUrl("https", "com.networknt.ab-1.0.0", null, null);
                connectionAb = client.connect(new URI(abHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        if(connectionAc == null || !connectionAc.isOpen()) {
            try {
                acHost = cluster.serviceToUrl("https", "com.networknt.ac-1.0.0", null, null);
                connectionAc = client.connect(new URI(acHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        if(connectionAd == null || !connectionAd.isOpen()) {
            try {
                adHost = cluster.serviceToUrl("https", "com.networknt.ad-1.0.0", null, null);
                connectionAd = client.connect(new URI(adHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final CountDownLatch latch = new CountDownLatch(3);
        final AtomicReference<ClientResponse> referenceAb = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceAc = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceAd = new AtomicReference<>();
        try {
            ClientRequest requestAb = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestAb, exchange);
            connectionAb.sendRequest(requestAb, client.createClientCallback(referenceAb, latch));

            ClientRequest requestAc = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestAc, exchange);
            connectionAc.sendRequest(requestAc, client.createClientCallback(referenceAc, latch));

            ClientRequest requestAd = new ClientRequest().setMethod(Methods.GET).setPath(path);
            if(securityEnabled) client.propagateHeaders(requestAd, exchange);
            connectionAd.sendRequest(requestAd, client.createClientCallback(referenceAd, latch));

            latch.await();

            int statusCodeAb = referenceAb.get().getResponseCode();
            if(statusCodeAb >= 300){
                throw new Exception("Failed to call API AB: " + statusCodeAb);
            }
            List<String> abList = Config.getInstance().getMapper().readValue(referenceAb.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(abList);

            int statusCodeAc = referenceAc.get().getResponseCode();
            if(statusCodeAc >= 300){
                throw new Exception("Failed to call API AC: " + statusCodeAc);
            }
            List<String> acList = Config.getInstance().getMapper().readValue(referenceAc.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(acList);

            int statusCodeAd = referenceAd.get().getResponseCode();
            if(statusCodeAd >= 300){
                throw new Exception("Failed to call API AD: " + statusCodeAd);
            }
            List<String> adList = Config.getInstance().getMapper().readValue(referenceAd.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(adList);

        } catch (Exception e) {
            logger.error("Exception:", e);
            throw new ClientException(e);
        }
        list.add("API AA: Message 1");
        list.add("API AA: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}
```

API AA needs to use service discovery to lookup API AB, AC and AD in order to call them. Let's put the
following config into service.yml to setup direct registry. On production, it can be changed to Consul
service registry and discovery without touching the code. Please refer to [service discovery][] for more
details. 


```yaml
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.ab-1.0.0: https://localhost:7442
        com.networknt.ac-1.0.0: https://localhost:7443
        com.networknt.ad-1.0.0: https://localhost:7444
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```


Start API A server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/openapi/ms-aggregate/aa/https
mvn clean install exec:exec
```
From another terminal window run:

```
curl -k https://localhost:7441/v1/data
```
And the result is

```
["API AB: Message 1","API AB: Message 2","API AC: Message 1","API AC: Message 2","API AD: Message 1","API AD: Message 2","API AA: Message 1","API AA: Message 2"]
```

At this moment, we have all four APIs completed and AA is calling AB, AC and AD using HTTPS connections.
In the next step let's test the [performance][] with wrk. 

[service discovery]: /tutorial/common/discovery/
[performance]: /tutorial/rest/openapi/ms-aggregate/performance/
