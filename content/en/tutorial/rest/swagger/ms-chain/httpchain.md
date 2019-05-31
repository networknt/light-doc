---
title: "Httpchain"
date: 2017-11-29T13:42:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Now these APIs are working if you start them and they will output the mock responses
generated based on the API specifications. Let's take a look at the API handler itself
and update it based on our business logic, you can call API A and subsequently all other
APIs will be called in a chain.

#### Prepare Environment

Before starting this step, let's create a folder called httpchain in each sub folder under
ms_chain and copy everything from generated folder to the httpchain. We are going to update
httpchain folder to have business logic to call another api and change the configuration
to listen to different port. You can compare between generated and httpchain to see what has
been changed later on.

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a
cp -r generated httpchain
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b
cp -r generated httpchain
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c
cp -r generated httpchain
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d
cp -r generated httpchain

```

Now we have httpchain folder copied from generated and all updates in this step will be
in httpchain folder. 

#### API D
Let's take a look at the PathHandlerProvider.java in
ms_chain/api_d/httpchain/src/main/java/com/networknt/apid

```
package com.networknt.apid;

import com.networknt.config.Config;
import com.networknt.handler.HandlerProvider;
import io.undertow.Handlers;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import com.networknt.info.ServerInfoGetHandler;
import com.networknt.health.HealthGetHandler;
import com.networknt.apid.handler.*;

public class PathHandlerProvider implements HandlerProvider {
    @Override
    public HttpHandler getHandler() {
        return Handlers.routing()
        
            .add(Methods.GET, "/v1/data", new DataGetHandler())
        
            .add(Methods.GET, "/v1/health", new HealthGetHandler())
        
            .add(Methods.GET, "/v1/server/info", new ServerInfoGetHandler())
        
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
in swagger specification. Here is the generated handler code. 
 
```

package com.networknt.apid.handler;

import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class DataGetHandler implements HttpHandler {
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
             exchange.getResponseSender().send(" [\n                                \"Message 1\",\n                                \"Message 2\"\n                            ]");
        
    }
}
```

Let's update it to an array of strings that indicates the response comes from API D. 


```
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

Now, let's build it and start the server. 
```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d/httpchain
mvn clean install exec:exec
```
Test it with curl.

```
curl localhost:7004/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2"]
```

Test with HTTPS port and you should have the same result.

```
curl -k https://localhost:7444/v1/data
```

Note that we have -k option to in the https command line as we are using self-signed
certificate and we don't want to verify the domain.


#### API C
Let's leave API D running and update API C DataGetHandler in 
~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/httpchain


```
package com.networknt.apic.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static String CONFIG_NAME = "api_c";
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static String apidHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_d_host");
    static String apidPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_d_path");
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connection;

    public DataGetHandler() {
        try {
            connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
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
                connection = client.connect(new URI(apidHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(apidPath);
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
        list.add("API C: Message 1");
        list.add("API C: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}
```

API C needs to have the url of API D in order to call it. Let's put it in a config file for
now and move to service discovery later.

Create api_c.yml in src/main/resources/config folder.

```
api_d_host: h2c-prior://localhost:7004
api_d_path: /v1/data
```

Note as we are using HTTP 2.0 to connect to API D and we know API D is HTTP 2.0 enabled. The
host protocol is h2c-prior instead of http to all Http2Client to use HTTP 2.0 to connect.

Start API C server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/httpchain
mvn clean install exec:exec
```
From another terminal window run:

```
curl localhost:7003/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2"]
```

Access https port should have the same result.

```
curl -k https://localhost:7443/v1/data

```

#### API B

Let's keep API C and API D running. The next step is to complete API B. API B 
will call API C to fulfill its request. 

Now let's update the generated DataGetHandler.java to the following in httpchain folder

```
package com.networknt.apib.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static String CONFIG_NAME = "api_b";
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static String apicHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_host");
    static String apicPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_path");
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connection;

    public DataGetHandler() {
        try {
            connection = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
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
                connection = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(apicPath);
            // this is to ask client module to pass through correlationId and traceabilityId as well as
            // getting access token from oauth2 server automatically and attatch authorization headers.
            if(securityEnabled) client.propagateHeaders(request, exchange);
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            if(statusCode >= 300){
                throw new Exception("Failed to call API C: " + statusCode);
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

API B needs to have the url of API C in order to call it. Let's put it in a config file for
now and move to service discovery later.

Create api_b.yml in src/main/resources/config folder.

```
api_c_host: h2c-prior://localhost:7003
api_c_path: /v1/data
```


Start API B server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b/httpchain
mvn clean install exec:exec
```
From another terminal window run:

```
curl localhost:7002/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2"]
```

Here is the https port and the result is the same.

```
curl -k https://localhost:7442/v1/data

```


#### API A

API A will call API B to fulfill its request. Now let's update the 
generated DataGetHandler.java code to

```
package com.networknt.apia.handler;

import com.fasterxml.jackson.core.type.TypeReference;
import com.networknt.client.Http2Client;
import com.networknt.config.Config;
import com.networknt.exception.ClientException;
import com.networknt.security.JwtHelper;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

public class DataGetHandler implements HttpHandler {
    static String CONFIG_NAME = "api_a";
    static Logger logger = LoggerFactory.getLogger(DataGetHandler.class);
    static String apibHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_b_host");
    static String apibPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_b_path");
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connection;

    public DataGetHandler() {
        try {
            connection = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
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
                connection = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(apibPath);
            // this is to ask client module to pass through correlationId and traceabilityId as well as
            // getting access token from oauth2 server automatically and attatch authorization headers.
            if(securityEnabled) client.propagateHeaders(request, exchange);
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            if(statusCode >= 300){
                throw new Exception("Failed to call API B: " + statusCode);
            }
            List<String> apidList = Config.getInstance().getMapper().readValue(reference.get().getAttachment(Http2Client.RESPONSE_BODY),
                    new TypeReference<List<String>>(){});
            list.addAll(apidList);
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


API A needs to have the url of API in order to call it. Let's put it in a config file for
now and move to service discovery later.

Create api_a.yml in src/main/resources/config folder.

```
api_b_host: h2c-prior://localhost:7002
api_b_path: /v1/data
```


Start API A server and test the endpoint /v1/data

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/httpchain
mvn clean install exec:exec
```
From another terminal window run:

```
curl localhost:7001/v1/data
```
And the result is

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

The https port and the result should be the same.

```
 curl -k https://localhost:7441/v1/data
```
At this moment, we have all four APIs completed and A is calling B, B is calling C and
C is calling D using Http connections.

In the [next step][], we are going to test the performance for HTTP connections. 


[next-step]: /tutorial/rest/swagger/ms-chain/httpperf/
