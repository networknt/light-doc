---
title: "Plugin Architecture"
date: 2017-11-06T11:14:26-05:00
description: "Plugin Architecture"
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

In the framework design, we have adopted the same principle of microservices architecture to break down the entire framework into smaller pieces so that each can be customized and replaced if necessary. The only difference is that all the modules in the framework are wired in the request/response chain for the best performance.

![component](/images/light_java_component.png)

There are four type of components that can be wired in at different stage of the server start up. The
following is the loading sequence.

* Startup Hook Providers
* Shutdown Hook Providers
* Handler Provider
* Middleware Handlers

![startup_sequence](/images/startup_sequence.png)

### Plugin implementation

There are so many ways to implement plugins and wire in different implementations of the same interface. Spring is one of the popular ways. However, it will make our framework dependent on a version of spring framework and that can cause a lot of problems if the API handlers are using different versions of spring framework. Also, we don’t want to make the framework depend on spring and force everyone to include it. In the end, we are using Java SPI (Service Provider Interface). In your generated API project, you can find four files in /src/main/resources/META-INF/services. These files contain plugins to be loaded/executed during server startup and shutdown.

### Shutdown Hook Providers

Shutdown hook providers are plugins that would be called during server shutdown in order to release
resources. For example, close database connections.

All shutdown hook providers will implement interface com.networknt.server.ShutdownHookProvider

```
public interface ShutdownHookProvider {
    void onShutdown();
}

```
The onShutdown() in your implementation will be called before server shutdown.

# Startup Hook Providers

Startup hook providers are plugins that would be called during server startup in order to initialize
resources. For example, create database connection pool, load spring application context etc.

All startup hook providers will implement interface com.networknt.server.StartupHookProvider

```
public interface StartupHookProvider {
    void onStartup();
}
```

The onStartup() in your implementation will be called before server startup.
 
### Handler Provider

There is only one handler provider that is needed to wire in API implementations. In most cases, it would be an instance of io.undertow.server.RoutingHandler just like the generated [petstore project](https://github.com/networknt/light-example-4j/tree/master/petstore). However, it is not limited and it can be several handlers chained together. One example is the  [webserver example](https://github.com/networknt/light-example-4j/tree/master/webserver) which has several handlers chained together to provide API routing as well as static websites. The handler provider code can be found [here](https://github.com/networknt/light-example-4j/blob/master/webserver/src/main/java/com/networknt/webserver/handler/WebServerHandlerProvider.java)

If you have OpenAPI specification defined, this handler provider will be generated from 
[light-codegen](https://github.com/networknt/light-codegen). [Here](https://github.com/networknt/light-example-4j/blob/master/petstore/src/main/java/io/swagger/handler/PathHandlerProvider.java) 
is a generated petstore handler provider that has the mapping for all endpoints.

The above is using light-rest-4j as example, for GraphQL and Hybrid, the handler provider will be
different.

Handler provider implements interface com.networknt.handler.HandlerProvider

```
public interface HandlerProvider {
    HttpHandler getHandler();
}

```

The getHandler() will return an HttpHandler or a chain of HttpHandlers wrapped together. This handler
will be called in the request/response chain right after all middleware handlers are called.


### Middleware Handlers

There are some [built-in middleware handlers][] in the framework to address common cross cutting 
concerns. There are implemented in a way we think the best to meet most of business requirements. 
In other words, there are opinionated. For product that built top of the framework, you can 
add/customize/replace existing middleware handlers. 

All middleware handlers need to implement interface com.networknt.handler.MiddlewareHandler.

```
public interface MiddlewareHandler extends HttpHandler {

    HttpHandler getNext();

    MiddlewareHandler setNext(final HttpHandler next);

    boolean isEnabled();

    void register();

}
```

boolean isEnabled() 

Every middleware handler has a corresponding config file which is the same name but in lowercase with a .json extension. There must be a flag to indicate if the handle will be wired in during server startup. This gives users a chance to temporarily disable a particular middleware handler without changing the SPI configuration.

void register()

This function will be called when the middleware is wired in the request/response chain. It registers itself and its configuration to the server info component which can be accessed through a special endpoint to get runtime information on middleware handlers and their configuration along with other system info.

MiddlewareHandler setNext(final HttpHandler next)

As middleware handlers are chained together, so the existing handler must be put in the current handler next. When the current handler is completed, it will call the next handler if there is no error. And eventually, the user provided handler will be called if all middleware handlers are completed without an error.

### The sequence of middleware handlers.

There are dependencies between middleware handlers so the sequence to plug them in is very important.

For default plugins generated from [light-codegen](https://github.com/networknt/light-codegen),
please refer to [Server][]

### Diagram

![handler_chain](/images/handler_chain.png)

Note that audit, metrics and exceptions need to be hooked in the response in order to handle exceptions on both the request and response phase, calculate response time, and dump response info into the audit.

[built-in middleware handlers]: /concern/
[Server]: /concern/server/