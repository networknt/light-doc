---
date: 2017-10-17T18:12:46-04:00
title: Static Configuration
---

Now we have four standalone services and the next step is to connect them together.

Here is the call tree for these services.

API A will call API B and API C to fulfill its request. API B will call API D
to fulfill its request.

```
API A -> API B -> API D
      -> API C
```

Before we change the code, let's copy the generated projects to new folders so
that we can compare the changes later on.

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r generated static
cd ~/networknt/light-example-4j/discovery/api_b
cp -r generated static
cd ~/networknt/light-example-4j/discovery/api_c
cp -r generated static
cd ~/networknt/light-example-4j/discovery/api_d
cp -r generated static
```

Let's start update the code in static folders for each project. If you are
using Intellij IDEA Community Edition, you need to open light-example-4j
repo and then import each project by right click pom.xml and select Open as
maven project in each static folder.

As indicated from the title, here we are going to hard code urls in API to API
calls in configuration files. That means these services will be deployed on the 
known hosts with known ports. And we will have a config file for each project to 
define the calling service urls.

### API A

For API A, as it is calling API B and API C, its handler needs to be changed to
call two other APIs and needs to load a configuration file that define the 
urls for API B and API C.

DataGetHandler.java

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
    static String apicHost = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_host");
    static String apicPath = (String) Config.getInstance().getJsonMapConfig(CONFIG_NAME).get("api_c_path");
    static Map<String, Object> securityConfig = (Map)Config.getInstance().getJsonMapConfig(JwtHelper.SECURITY_CONFIG);
    static boolean securityEnabled = (Boolean)securityConfig.get(JwtHelper.ENABLE_VERIFY_JWT);
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connectionB;
    static ClientConnection connectionC;

    public DataGetHandler() {
        try {
            connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exeption:", e);
        }
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        List<String> list = new ArrayList<>();
        if(connectionB == null || !connectionB.isOpen()) {
            try {
                connectionB = client.connect(new URI(apibHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        if(connectionC == null || !connectionC.isOpen()) {
            try {
                connectionC = client.connect(new URI(apicHost), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            } catch (Exception e) {
                logger.error("Exeption:", e);
                throw new ClientException(e);
            }
        }
        final CountDownLatch latch = new CountDownLatch(2);
        final AtomicReference<ClientResponse> referenceB = new AtomicReference<>();
        final AtomicReference<ClientResponse> referenceC = new AtomicReference<>();
        try {
            ClientRequest requestB = new ClientRequest().setMethod(Methods.GET).setPath(apibPath);
            if(securityEnabled) client.propagateHeaders(requestB, exchange);
            connectionB.sendRequest(requestB, client.createClientCallback(referenceB, latch));

            ClientRequest requestC = new ClientRequest().setMethod(Methods.GET).setPath(apicPath);
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

The following is the config file that define the url for API B and API C. This is hard
coded and can only be changed in this config file and restart the server. For now, I
am just creating the file in src/main/resources/config folder, but it should be 
externalized on official environment.

api_a.yml

```
api_b_host: https://localhost:7442
api_b_path: /v1/data
api_c_host: https://localhost:7443
api_c_path: /v1/data
```

### API B

Change the handler to call API D and load configuration for API D url.

DataGetHandler.java

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
        list.add("API B: Message 1");
        list.add("API B: Message 2");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(list));
    }
}
```

Configuration file for API D url.

api_b.yml

```
api_d_host: https://localhost:7444
api_d_path: /v1/data
```

### API C


Update API C handler to return information that associates with API C.

DataGetHandler.java

```
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

Update Handler for API D to return messages related to API D.

DataGetHandler.java

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


### Start Servers

Now let's start all four servers from four terminals.

API A

```
cd ~/networknt/light-example-4j/discovery/api_a/static
mvn clean install exec:exec
```

API B

```
cd ~/networknt/light-example-4j/discovery/api_b/static
mvn clean install exec:exec

```

API C

```
cd ~/networknt/light-example-4j/discovery/api_c/static
mvn clean install exec:exec

```

API D

```
cd ~/networknt/light-example-4j/discovery/api_d/static
mvn clean install exec:exec

```

When starting servers in above sequence, you can see some errors in API A and B as
the target server are not up yet. However, once you issue a call to API A, the
connections will be established immediately and then cached.

### Test Servers

Let's access API A and see if we can get messages from all four servers.

```
curl http://localhost:7001/v1/data

```
The result is 

```
["API C: Message 1","API C: Message 2","API D: Message 1","API D: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```

For now we have four APIs updated and with configured host and path, API A can call API B, C 
and API B can call API D. As you can see we are using https connection between API calls. 

In the next step, we are going to switch our implementation to [dynamic][] discovery with service.yml

[dynamic]: /tutorial/common/discovery/dynamic/
