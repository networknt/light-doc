---
title: "View Service"
date: 2018-01-08T10:04:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is a query side microservice implemented with light-rest-4j and generated from config
and OpenAPI 3.0 specification located at https://github.com/networknt/model-config/tree/master/rest/tram-todo/multi-module/view-service

For more details on how to use light-codegen, please refer to [light-codegen tutorial][]

As query side or view side needs to subscribe the events sent from command side, we need to
have a StartupHookProvider implementation to configure DomainEventDispatcher. 

StartUpTramMessageDispatcher.java

```java
package com.networknt.tram.todolist.view.startup;

import com.networknt.server.StartupHookProvider;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.event.subscriber.DomainEventDispatcher;
import com.networknt.tram.message.consumer.MessageConsumer;
import com.networknt.tram.todolist.view.TodoEventConsumer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class StartUpTramMessageDispacher implements StartupHookProvider {

	static final Logger logger = LoggerFactory.getLogger(StartUpTramMessageDispacher.class);
	public static  DomainEventDispatcher domainEventDispatcher;

	@Override
	public void onStartup() {
		MessageConsumer messageConsumer = SingletonServiceFactory.getBean(MessageConsumer.class);
		TodoEventConsumer todoEventConsumer = SingletonServiceFactory.getBean(TodoEventConsumer.class);
		domainEventDispatcher = new DomainEventDispatcher("todoServiceEvents", todoEventConsumer.domainEventHandlers(), messageConsumer);
		domainEventDispatcher.initialize();
	}
}

```

In the startup dispatcher, it uses MessageConsumer implementation and TodoEventConsumer which
we have to implement in this module. Here is the code. 

In this module, there is only one TodoviewGetHandler.

```java
package com.networknt.tram.todolist.view.handler;

import com.networknt.config.Config;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.todolist.view.TodoView;
import com.networknt.tram.todolist.view.TodoViewService;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import java.util.List;


public class TodoviewsGetHandler implements HttpHandler {
    private TodoViewService todoViewService =  (TodoViewService) SingletonServiceFactory.getBean(TodoViewService.class);

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        System.out.println("start call view service...");
        String searchValue = exchange.getQueryParameters().get("searchValue").getFirst();
        System.out.println("value->: " + searchValue);

        List<TodoView> todos =  todoViewService.search(searchValue);
        System.out.println("size:" + todos);

        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(todos));

    }
}

```

And a startup hook handler.

```java
package com.networknt.tram.todolist.view.startup;

import com.networknt.server.StartupHookProvider;
import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.event.subscriber.DomainEventDispatcher;
import com.networknt.tram.message.consumer.MessageConsumer;
import com.networknt.tram.todolist.view.TodoEventConsumer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class StartUpTramMessageDispacher implements StartupHookProvider {

	static final Logger logger = LoggerFactory.getLogger(com.networknt.tram.todolist.startup.StartUpTramMessageDispacher.class);
	public static  DomainEventDispatcher domainEventDispatcher;

	@Override
	public void onStartup() {
		MessageConsumer messageConsumer = SingletonServiceFactory.getBean(MessageConsumer.class);
		TodoEventConsumer todoEventConsumer = SingletonServiceFactory.getBean(TodoEventConsumer.class);
		domainEventDispatcher = new DomainEventDispatcher("todoServiceEvents", todoEventConsumer.domainEventHandlers(), messageConsumer);
		domainEventDispatcher.initialize();
	}
}

```

In the next step, let's [build and test][] both command side service and query side service together. 

[light-codegen tutorial]: /tutorial/generator/
[build and test]: /tutorial/tram/todo-list/test/
