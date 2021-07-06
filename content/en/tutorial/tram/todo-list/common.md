---
title: "Common"
date: 2018-01-08T10:03:52-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This module contains events shared between command side and query side. Also it has a utility class
to generate UUID which will be used as key for events.

The Utils.java is very simple and it is included just to ensure that the common is self-contained
without too many dependencies.

```java
package com.networknt.tram.todolist.common;


import java.util.UUID;

public class Utils {
  public static String generateUniqueString() {
    return UUID.randomUUID().toString();
  }
}

```

The following are three domain events class that implement DomainEvent interface.

TodoCreated.java

```java
package com.networknt.tram.todolist.common;


import com.networknt.tram.event.common.DomainEvent;

public class TodoCreated implements DomainEvent {

  private String title;

  private boolean completed;

  private int executionOrder;

  public TodoCreated() {
  }

  public TodoCreated(String title, boolean completed, int executionOrder) {
    this.title = title;
    this.completed = completed;
    this.executionOrder = executionOrder;
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

TodoDeleted.java

```java
package com.networknt.tram.todolist.common;


import com.networknt.tram.event.common.DomainEvent;

public class TodoDeleted implements DomainEvent {
}

```

TodoUpdated.java

```java
package com.networknt.tram.todolist.common;


import com.networknt.tram.event.common.DomainEvent;

public class TodoUpdated implements DomainEvent {

  private String title;

  private boolean completed;

  private int executionOrder;

  public TodoUpdated() {
  }

  public TodoUpdated(String title, boolean completed, int executionOrder) {
    this.title = title;
    this.completed = completed;
    this.executionOrder = executionOrder;
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

As you can see these are just POJO and DomainEvent doesn't have any method but just and identifier.

The next step, we are going to walk through [command][] side common components. 

[command]: /tutorial/tram/todo-list/command/
