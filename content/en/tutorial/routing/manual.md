---
title: "Manual Routing"
date: 2019-08-28T22:32:50-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Sometimes, you need the maximum flexibility in defining the routing for different combinations of paths, methods, and parameters. You can use the `Handlers.routing()` builder to add routing rules in the source code. 

The following is an example to show you how to use undertow routing handler. It opens the door to customize it and put your handlers in front of common handlers provided. It is a very low-level implementation on top of the light-4j framework.

```
package com.networknt.example.handler;

import com.networknt.config.Config;
import com.networknt.handler.HandlerProvider;
import io.undertow.Handlers;
import io.undertow.predicate.Predicates;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.server.handlers.PathHandler;
import io.undertow.server.handlers.PathTemplateHandler;
import io.undertow.util.Methods;


public class ExampleHandlerProvider implements HandlerProvider {

    public HttpHandler getHandler() {
        HttpHandler handler = Handlers.routing()
                .add(Methods.GET, "/baz", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("baz");
                    }
                })
                .add(Methods.GET, "/baz/{foo}", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("baz-path" + exchange.getQueryParameters().get("foo"));
                    }
                })
                .add(Methods.GET, "/bar", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("get bar");
                    }
                })
                .add(Methods.PUT, "/bar", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("put bar");
                    }
                })
                .add(Methods.POST, "/bar", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("post bar");
                    }
                })
                .add(Methods.DELETE, "/bar", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("delete bar");
                    }
                })
                .add(Methods.GET, "/wild/{test}/*", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("wild:" + exchange.getQueryParameters().get("test") + ":" + exchange.getQueryParameters().get("*"));
                    }
                })
                .add(Methods.GET, "/foo", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("foo");
                    }
                })
                .add(Methods.GET, "/foo", Predicates.parse("contains[value=%{i,SomeHeader},search='special'] "), new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("special foo");
                    }
                })
                .add(Methods.POST, "/foo", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("posted foo");
                    }
                })
                .add(Methods.GET, "/foo/{bar}", new HttpHandler() {
                    public void handleRequest(HttpServerExchange exchange) throws Exception {
                        exchange.getResponseSender().send("foo-path" + exchange.getQueryParameters().get("bar"));
                    }
                });
        return handler;
    }
}

```

The source code repo can be found at [routing repository][], and you can run the following command to build and start it. 

```
mvn clean install exec:exec
```

To test it, you can take a look at the source code and issue the correct curl command, for example. 


```
curl -k https://localhost:8443/bar
```

And the result is 

```
get bar
```

[routing repository]: https://github.com/networknt/light-example-4j/tree/master/routing

