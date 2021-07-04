---
title: "Petstore Jaeger"
date: 2019-06-15T22:40:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have started the [all-in-one][] Jaeger. You can see that there is only one service registered as jaeger-query on the UI. In this step, we are going to change the configuration of petstore to enable jaeger-tracing. 

### petstore

First, let's make sure that the petstore application is working in the light-example-4j. If you haven't checked out the light-example-4j, please do so.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j
mvn clean install exec:exec
```

The server should be up and running now. Let's issue a curl command to test it.

```
curl -k https://localhost:8443/v1/pets
```

The result should be. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

### petstore-jaeger

Once the server is confirmed working, we are going to copy the petstore to a new folder called petstore-jaeger as we need to modify the petstore a little bit for jaeger-tracing. 

```
cd ~/networknt/light-example-4j/rest/openapi
cp -r petstore petstore-jaeger
```

We need to update the pom.xml in petstore-jaeger to add dependency for jaeger-tracing module.

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>jaeger-tracing</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

Next, we will create a config folder in light-config-test to add tracing to the application. Before doing that, we need to build a fatjar for petstore-jaeger so that we can start the server with `java -jar -Dlight-4j-config-dir=`

To speed up the development cycle, all generated light-4j projects will only be built into a samll jar that can be started with `exec:exec` target in Maven. This is to avoid building source.jar doc.jar and fatjar which takes a lot of time during active development. When you need to build a fatjar, you can use the `release` profile. 

```
cd ~/networknt/light-example-4j/rest/openapi/petstore-jaeger
mvn clean install -Prelease
```

Now you should find a fatjar about 12MB in the target folder called `petstore-3.0.1.jar`

### config

Next, we are going to create a config folder for petstore-jaeger to replace traceability and correlation with jaeger-tracing cross-cutting concerns in handler.yml and update service.yml to add JaegerStartupHookProvider. 

If you haven't checked out light-config-test repo, you can do it now. 

```
cd ~/networknt
git clone https://github.com/networknt/light-config-test.git
```

You can find the config files in `~/networknt/light-config-test/light-example-4j/rest/openapi/petstore-jaeger/config`

##### handler.yml

The handler.yml is copied from the petstore-jaeger /src/main/resources/config folder and we have made the following modifications. 

In the handlers section, we have commented out the traceability and correlation handlers and added jaeger handler. 

```
  # - com.networknt.traceability.TraceabilityHandler@traceability
  # - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.jaeger.tracing.JaegerHandler@jaeger

```

In the default chain section, we commented out the traceability and correlation and added jaeger. 

```
    # - traceability
    # - correlation
    - jaeger
```

##### service.yml

The service.yml is copied as well. And we have added the following section. 

```
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  # If you are using mask module to remove sensitive info before logging, uncomment the following line.
  - com.networknt.jaeger.tracing.JaegerStartupHookProvider
```

This will ensure that the Jaeger trace is initialized during server startup.


##### jaeger-tracing.yml

This is a new configuration file we added to control how jaeger-tracing module works. 

jaeger-tracing.yml

```
---
# Jaeger tracing implementation for the startup hook and handlers

# Indicate if the Jaeger tracing is enable or not.
enabled: true
# Sampler configuration https://github.com/jaegertracing/jaeger-client-java
type: const # can either be const, probabilistic, or ratelimiting
param: 1 # can either be an integer, a double, or an integer

```

### Test

Now, let's start the server with externalized config folder. 

```
cd ~/networknt
java -jar -Dlight-4j-config-dir=light-config-test/light-example-4j/rest/openapi/petstore-jaeger/config light-example-4j/rest/openapi/petstore-jaeger/target/petstore-3.0.1.jar

```

Once the server is up and running, issue the following command again. 

```
curl -k https://localhost:8443/v1/pets
```

Goto the Jaeger UI, you should find the trace for this request. 

![petstore-jaeger](/images/petstore-jaeger.png)

### handlers

Until now, we have not touched any existing code in petstore-jaeger after copying it from the petstore project. The only file that we have updated in the folder is the pom.xml that we have added the jaeger-tracing module into the dependency. 

From the above picture, you can see that the request trace is flat which means we have only one span per request. In this step, we are going to update the `PetsGetHandler` to add tracing to it. 

Here is the current handleRequest method. 

```
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.setStatusCode(200);
        exchange.getResponseSender().send("[{\"id\":1,\"name\":\"catten\",\"tag\":\"cat\"},{\"id\":2,\"name\":\"doggy\",\"tag\":\"dog\"}]");
    }

```

Let's change it to.

```
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        Span span = JaegerStartupHookProvider.tracer.buildSpan("PetsGetHandler").asChildOf(exchange.getAttachment(JaegerHandler.ROOT_SPAN)).start();
        try {
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
            exchange.setStatusCode(200);
            exchange.getResponseSender().send("[{\"id\":1,\"name\":\"catten\",\"tag\":\"cat\"},{\"id\":2,\"name\":\"doggy\",\"tag\":\"dog\"}]");
        } finally {
            span.finish();
        }
    }
```

As you can see that we get the trace object from the JaegerStartupHookProvider and create a span from the root span from the exchange attachment. 

Now try the same curl command above and go the UI to explore the request. You now can see something like this. 

![petstore-jaeger-handler](/images/petstore-jaeger-handler.png)

This time, you can see the PetsGetHandler span is nested in the /v1/pets@GET span. 


### Sync Call

Most times, you are not going to put all your business login in the handleRequest but scatter the logic into several classes or methods. In this step, we are going to explore the call from the same thread of handlerRequest. 

Let's create another class to simulate the database call in the handler package. 

PetsRepository.java

```
package com.networknt.petstore.handler;

import com.networknt.jaeger.tracing.JaegerStartupHookProvider;
import io.opentracing.Scope;
import io.opentracing.Span;

public class PetsRepository {
    public static String getPetById(String id) {
        Span span = JaegerStartupHookProvider.tracer.buildSpan("getPetByIdFromDb").asChildOf(JaegerStartupHookProvider.tracer.activeSpan()).start();
        try {
            try {Thread.sleep(10); } catch (InterruptedException e) {}
            span.log("database call returned");
            return "{\"id\":1,\"name\":\"Jessica Right\",\"tag\":\"pet\"}";
        } finally {
            span.finish();
        }
    }
}

```

Then let's update the PetsPetIdGetHandler.java

```
package com.networknt.petstore.handler;

import com.networknt.handler.LightHttpHandler;
import com.networknt.jaeger.tracing.JaegerHandler;
import com.networknt.jaeger.tracing.JaegerStartupHookProvider;
import io.opentracing.Span;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.HashMap;
import java.util.Map;

public class PetsPetIdGetHandler implements LightHttpHandler {
    
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        Span span = JaegerStartupHookProvider.tracer.buildSpan("PetsPetIdGetHandler").asChildOf(exchange.getAttachment(JaegerHandler.ROOT_SPAN)).start();
        JaegerStartupHookProvider.tracer.scopeManager().activate(span);
        try  {
            String petId = exchange.getPathParameters().get("petId").getFirst();
            span.setTag("petId", petId);
            String result = PetsRepository.getPetById(petId);
            exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
            exchange.setStatusCode(200);
            exchange.getResponseSender().send(result);
        } catch (Exception e) {
            span.log(e.getMessage());
        } finally {
            span.finish();
        }
    }
}

```

When you access the server with the folllowing command.

```
curl -k https://localhost:8443/v1/pets/111
```

You got the trace like this.

![petstore-jaeger-sync](/images/petstore-jaeger-sync.png)


### Async Call

Sometimes, we call another function in another thread, and the activeSpan in the ThreadLocal won't work anymore. The best way is to set the active Span in Trace and then in the thread, create a child span based on it. 


The other option is to use the Tracer in the JaegerStartupHookProvider to create new span as it is available all the time in the light-4j application. 


[all-in-one]: /tutorial/tracing/jaeger/all-in-one/
