---
title: "View"
date: 2018-01-08T10:04:02-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

TodoEventConsumer.java

```java
package com.networknt.tram.todolist.view;

import com.networknt.tram.event.subscriber.DomainEventHandlers;
import com.networknt.tram.event.subscriber.DomainEventHandlersBuilder;
import com.networknt.tram.todolist.command.Todo;
import com.networknt.tram.todolist.common.TodoCreated;
import com.networknt.tram.todolist.common.TodoDeleted;
import com.networknt.tram.todolist.common.TodoUpdated;


public class TodoEventConsumer {

  private TodoViewService todoViewService;

  public TodoEventConsumer(TodoViewService todoViewService) {
      this.todoViewService = todoViewService;
  }

  public DomainEventHandlers domainEventHandlers() {
    return DomainEventHandlersBuilder
            .forAggregateType(Todo.class.getName())
            .onEvent(TodoCreated.class, dee -> {
              TodoCreated todoCreated = dee.getEvent();
              todoViewService.index(new TodoView(dee.getAggregateId(),
                  todoCreated.getTitle(), todoCreated.isCompleted(), todoCreated.getExecutionOrder()));
            })
            .onEvent(TodoUpdated.class, dee -> {
              TodoUpdated todoUpdated = dee.getEvent();
              todoViewService.index(new TodoView(dee.getAggregateId(),
                  todoUpdated.getTitle(), todoUpdated.isCompleted(), todoUpdated.getExecutionOrder()));
            })
            .onEvent(TodoDeleted.class, dee ->
                    todoViewService.remove(dee.getAggregateId()))
            .build();
  }
}

```

As you can see that three events are handled in the handler to call TodoViewService to update
the materialized view. Here is the TodoViewService interface. 

TodoViewService.java

```java
package com.networknt.tram.todolist.view;

import java.util.List;

public interface TodoViewService {

  List<TodoView> search(String value);

  void index(TodoView todoView) ;

  void remove(String id) ;

}

```

Above interface uses a TodoView object and here is the source code.

TodoView.java

```java
package com.networknt.tram.todolist.view;

public class TodoView {

  public static final String INDEX = "todos";
  public static final String TYPE = "todo";

  private String id;

  private String title;

  private boolean completed;

  private int executionOrder;

  public TodoView() {
  }

  public TodoView(String id, String title, boolean completed, int executionOrder) {
    this.id = id;
    this.title = title;
    this.completed = completed;
    this.executionOrder = executionOrder;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getTitle() {
    return title;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public boolean isCompleted() {
    return completed;
  }

  public void setCompleted(boolean completed) {
    this.completed = completed;
  }

  public int getExecutionOrder() {
    return executionOrder;
  }

  public void setExecutionOrder(int executionOrder) {
    this.executionOrder = executionOrder;
  }
}

```

The following is the implementation of TodoViewService interface.

TodoViewServiceImpl.java

```java
package com.networknt.tram.todolist.view;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.transport.client.PreBuiltTransportClient;

import java.io.IOException;
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;


public class TodoViewServiceImpl implements TodoViewService {

  private static TransportClient transportClient;

  private ObjectMapper objectMapper = new ObjectMapper();

  public TodoViewServiceImpl() {

  }

  public TodoViewServiceImpl(String host, int port) {
    try {
      transportClient =  new PreBuiltTransportClient(Settings.builder().put("client.transport.ignore_cluster_name", true).build())
              .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName(host), port));
    } catch (Exception e) {
      //TODO
    }
  }

 @Override
  public List<TodoView> search(String value) {

    if (!transportClient.admin().indices().prepareExists(TodoView.INDEX).execute().actionGet().isExists()) {
      return Collections.emptyList();
    }

    SearchResponse response = transportClient.prepareSearch(TodoView.INDEX)
            .setTypes(TodoView.TYPE)
            .setQuery(QueryBuilders.termQuery("_all", value))
            .get();

    List<TodoView> result = new ArrayList<>();

    for (SearchHit searchHit : response.getHits()) {
      try {
        result.add(objectMapper.readValue(searchHit.getSourceAsString(), TodoView.class));
      }
      catch (IOException e) {
        throw new RuntimeException(e);
      }
    }

    return result;
  }

  @Override
  public void index(TodoView todoView) {

      System.out.println("view side-------------------->>> " + todoView);
    try {
      IndexResponse ir = transportClient
          .prepareIndex(TodoView.INDEX, TodoView.TYPE, todoView.getId())
          .setSource(objectMapper.writeValueAsString(todoView), XContentType.JSON)
          .get();
    }
    catch (JsonProcessingException e) {
      throw new RuntimeException(e);
    }
  }

  @Override
  public void remove(String id) {
    transportClient.prepareDelete(TodoView.INDEX, TodoView.TYPE, id).get();
  }
}

```

This service implementation is using ElesticSearch to index the TodoView so that we can search
the object and update the index once it receives command side events.

In the next step, let's take a look at [tram-todo-command][] restful microservice. 

[tram-todo-command]: /tutorial/tram/todo-list/command-service/
