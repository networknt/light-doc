---
title: "Command"
date: 2018-01-08T10:03:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This module contains command side repository and service implementation and it is a foundation
of command side restful service. 

First let's take a look at Todo class which is a data object. 

Todo.java

```java
package com.networknt.tram.todolist.command;


import com.networknt.eventuate.jdbc.IdGenerator;
import com.networknt.eventuate.jdbc.IdGeneratorImpl;

public class Todo {

  private static IdGenerator idGenerator = new IdGeneratorImpl();
  private String id;

  private String title;

  private boolean completed;

  private int executionOrder;

  public Todo() {
  }

  public Todo(String title, boolean completed, int executionOrder) {
    this.id = idGenerator.genId().asString();
    this.title = title;
    this.completed = completed;
    this.executionOrder = executionOrder;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getId() {
    return id;
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

We need a repository interface to save Todo object into database. 

TodoRepository.java

```java
package com.networknt.tram.todolist.command;


import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;

public interface TodoRepository {

    void setConnection(Connection connection);

    List<Todo>  getAll();

    Todo findOne(String id);

    Todo save(Todo todo) throws SQLException;

    Todo update(Todo todo) throws TodoNotFoundException, SQLException;

    void delete(String id) throws SQLException;
}

```

And the implementation for the interface is here.

TodoRepositoryImpl.java

```java
package com.networknt.tram.todolist.command;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class TodoRepositoryImpl implements TodoRepository {

    protected Logger logger = LoggerFactory.getLogger(getClass());

    private Connection connection;

    public TodoRepositoryImpl() {

    }

    public TodoRepositoryImpl(Connection connection) {
        this.connection = connection;
    }

    public void setConnection(Connection connection) {
        this.connection = connection;
    }


    @Override
    public List<Todo> getAll() {
        List<Todo> todos = new ArrayList<>();

        try {
            String psSelect = "SELECT ID, TITLE, COMPLETED, ORDER_ID FROM todo_db.TODO WHERE ACTIVE_FLG = 'Y' order by ORDER_ID asc";
            PreparedStatement stmt = connection.prepareStatement(psSelect);
            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                Todo todo = new Todo();
                todo.setTitle(rs.getString("ID"));
                todo.setTitle(rs.getString("TITLE"));
                todo.setCompleted(rs.getBoolean("COMPLETED"));
                todo.setExecutionOrder(rs.getInt("ORDER_ID"));
                todos.add(todo);
            }
        } catch (SQLException e) {
            logger.error("SqlException:", e);
        }

        return todos;
    }

    @Override
    public Todo findOne(String id) {
        Todo todo = null;
        try {
            String psSelect = "SELECT ID, TITLE, COMPLETED, ORDER_ID FROM todo_db.TODO WHERE ACTIVE_FLG = 'Y' AND ID = ?";
            PreparedStatement stmt = connection.prepareStatement(psSelect);
            stmt.setString(1, id);
            ResultSet rs = stmt.executeQuery();
            if (rs == null || rs.getFetchSize() > 1) {
                logger.error("incorrect fetch result {}", id);
            }
            while (rs.next()) {
                todo = new Todo();
                todo.setId(rs.getString("ID"));
                todo.setTitle(rs.getString("TITLE"));
                todo.setCompleted(rs.getBoolean("COMPLETED"));
                todo.setExecutionOrder(rs.getInt("ORDER_ID"));
            }
        } catch (SQLException e) {
            logger.error("SqlException:", e);
        }

        return todo;
    }

    @Override
    public Todo save(Todo todo) throws SQLException{
            String psInsert = "INSERT INTO todo_db.TODO (ID, TITLE, COMPLETED, ORDER_ID) VALUES (?, ?, ?, ?)";
            try (PreparedStatement stmt = connection.prepareStatement(psInsert)) {
                stmt.setString(1, todo.getId());
                stmt.setString(2, todo.getTitle());
                stmt.setBoolean(3, todo.isCompleted());
                stmt.setInt(4, todo.getExecutionOrder());
                int count = stmt.executeUpdate();
                if (count != 1) {
                    logger.error("Failed to insert TODO: {}", todo.getId());
                } else {
                    return todo;
                }
            } catch (SQLException e) {
                logger.error("SqlException:", e);
                throw new SQLException(e);
            }


            return null;
    }

    @Override
    public Todo update(Todo todo)  throws SQLException{
            String psUpdate = "UPDATE todo_db.TODO SET TITLE=?, COMPLETED=?, ORDER_ID=? WHERE ID=? ";
            try (PreparedStatement stmt = connection.prepareStatement(psUpdate)) {
                stmt.setString(1, todo.getTitle());
                stmt.setBoolean(2, todo.isCompleted());
                stmt.setInt(3, todo.getExecutionOrder());
                stmt.setString(4, todo.getId());
                int count = stmt.executeUpdate();
                if (count != 1) {
                    logger.error("Failed to update TODO: {}", todo.getId());
                } else {
                    return todo;
                }

            } catch (SQLException e) {
                logger.error("SqlException:", e);
                throw new SQLException(e);
            }


            return null;
    }

    @Override
    public void delete(String id)  throws SQLException{
            String psDelete = "UPDATE todo_db.TODO SET ACTIVE_FLG = 'N' WHERE ID = ?";
            try (PreparedStatement psEntity = connection.prepareStatement(psDelete)) {
                psEntity.setString(1, id);
                int count = psEntity.executeUpdate();
                if (count != 1) {
                    logger.error("Failed to delete TODO: {}", id);
                }
            } catch (SQLException e) {
                logger.error("SqlException:", e);
                throw new SQLException(e);
            }



    }


}

```

As you have noticed, the database connection is passed in from the constructor so that all db
operation will join a JDBC transaction with the same connection. 

The following is a service class that wires in both database actions together. Update todo object
and insert an event to message table in the same transaction. 

TodoCommandService.java

```java
package com.networknt.tram.todolist.command;

import com.networknt.eventuate.jdbc.IdGenerator;
import com.networknt.eventuate.jdbc.IdGeneratorImpl;
import com.networknt.tram.event.common.DomainEvent;
import com.networknt.tram.event.publisher.DomainEventPublisher;
import com.networknt.tram.event.publisher.DomainEventPublisherImpl;
import com.networknt.tram.message.producer.MessageProducer;
import com.networknt.tram.message.producer.jdbc.MessageProducerJdbcConnectionImpl;
import com.networknt.tram.todolist.common.TodoCreated;
import com.networknt.tram.todolist.common.TodoDeleted;
import com.networknt.tram.todolist.common.TodoUpdated;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

import static java.util.Arrays.asList;


public class TodoCommandService {

  protected Logger logger = LoggerFactory.getLogger(getClass());

  private TodoRepository todoRepository;

  private DomainEventPublisher domainEventPublisher;

  private IdGenerator idGenerator;

  private DataSource dataSource;

  public TodoCommandService( DataSource dataSource,IdGenerator idGenerator) {
    this.dataSource = dataSource;
    this.idGenerator = idGenerator;
  }

  public Todo create(CreateTodoRequest createTodoRequest) {

      Todo todo = new Todo(createTodoRequest.getTitle(), createTodoRequest.isCompleted(), createTodoRequest.getOrder());
      try (final Connection connection = dataSource.getConnection()) {
          connection.setAutoCommit(false);
          this.todoRepository = new TodoRepositoryImpl(connection);
          MessageProducer messageProducer = new MessageProducerJdbcConnectionImpl(connection,idGenerator);
          this.domainEventPublisher = new DomainEventPublisherImpl(messageProducer);

          todo = todoRepository.save(todo);
          publishTodoEvent(todo, new TodoCreated(todo.getTitle(), todo.isCompleted(), todo.getExecutionOrder()));

          connection.commit();
      } catch (SQLException e) {
          logger.error("SqlException:", e);
      }

      return todo;
  }

  private void publishTodoEvent(Todo todo, DomainEvent... domainEvents) {
    domainEventPublisher.publish(Todo.class, todo.getId(), asList(domainEvents));
  }

  private void publishTodoEvent(String id, DomainEvent... domainEvents) {
    domainEventPublisher.publish(Todo.class, id, asList(domainEvents));
  }

  public Todo update(String id, UpdateTodoRequest updateTodoRequest) throws TodoNotFoundException {
      try (final Connection connection = dataSource.getConnection()) {
          connection.setAutoCommit(false);
          this.todoRepository = new TodoRepositoryImpl(connection);
          Todo todo = todoRepository.findOne(id);
          if (todo == null) {
              throw new TodoNotFoundException(id);
          }
          todo.setTitle(updateTodoRequest.getTitle());
          todo.setCompleted(updateTodoRequest.isCompleted());
          todo.setExecutionOrder(updateTodoRequest.getOrder());
          todoRepository.update(todo);

          MessageProducer messageProducer = new MessageProducerJdbcConnectionImpl(connection, new IdGeneratorImpl());
          this.domainEventPublisher = new DomainEventPublisherImpl(messageProducer);
          publishTodoEvent(todo, new TodoUpdated(todo.getTitle(), todo.isCompleted(), todo.getExecutionOrder()));

          connection.commit();

          return todo;
      } catch (SQLException e) {
          logger.error("SqlException:", e);
      }

    return null;
  }

  public void delete(String id) {
      try (final Connection connection = dataSource.getConnection()) {
          this.todoRepository = new TodoRepositoryImpl(connection);
          MessageProducer messageProducer = new MessageProducerJdbcConnectionImpl(connection,idGenerator);
          this.domainEventPublisher = new DomainEventPublisherImpl(messageProducer);

          todoRepository.delete(id);
          publishTodoEvent(id, new TodoDeleted());

      } catch (SQLException e) {
          logger.error("SqlException:", e);
      }

  }

  public Todo findOne(String id) {
      Todo todo = null;
      try (final Connection connection = dataSource.getConnection()) {
          this.todoRepository = new TodoRepositoryImpl(connection);
          todo = todoRepository.findOne(id);

      } catch (SQLException e) {
          logger.error("SqlException:", e);
      }
      return todo;
  }
}

```

Pay attention on how connection is handled. First, the autocommit is turned off and the same connection
object is passed into TodoResposity and MessageProducer implementations. 


There is a unit test to ensure that above service is working. 

```java
package com.networknt.tram.todolist.command;



import com.networknt.service.SingletonServiceFactory;
import com.networknt.tram.todolist.common.Utils;
import org.h2.tools.RunScript;
import org.junit.Assert;
import org.junit.Test;

import javax.sql.DataSource;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.sql.Connection;
import java.sql.SQLException;


public class CommandModuleTest {
  public static DataSource ds;

  static {
    ds = SingletonServiceFactory.getBean(DataSource.class);
    try (Connection connection = ds.getConnection()) {
      // Runscript doesn't work need to execute batch here.
      String schemaResourceName = "/todolist-example-h2-ddl.sql";
      InputStream in = CommandModuleTest.class.getResourceAsStream(schemaResourceName);

      if (in == null) {
        throw new RuntimeException("Failed to load resource: " + schemaResourceName);
      }
      InputStreamReader reader = new InputStreamReader(in);
      RunScript.execute(connection, reader);

    } catch (SQLException e) {
      e.printStackTrace();
    }
  }

  private TodoCommandService todoCommandService = SingletonServiceFactory.getBean(TodoCommandService.class);

  @Test
  public void testCreate() {
    String title = Utils.generateUniqueString();
    String id = todoCommandService.create(new CreateTodoRequest(title, false, 0)).getId();
    Todo todo = todoCommandService.findOne(id);
    Assert.assertNotNull(todo);
    Assert.assertEquals(title, todo.getTitle());
  }

  @Test
  public void testUpdate() throws TodoNotFoundException {
    Todo todo = todoCommandService.create(new CreateTodoRequest(Utils.generateUniqueString(), false, 9));
    String title = Utils.generateUniqueString();
    todoCommandService.update(todo.getId(), new UpdateTodoRequest(title, false, 0));
    todo = todoCommandService.findOne(todo.getId());
    Assert.assertNotNull(todo);
    Assert.assertEquals(title, todo.getTitle());
  }

  @Test
  public void testDelete() {
    Todo todo = todoCommandService.create(new CreateTodoRequest(Utils.generateUniqueString(), false, 9));
    todoCommandService.delete(todo.getId());
    Assert.assertNull(todoCommandService.findOne(todo.getId()));
  }
}

```
 
The unit test is using H2 database and the script todolist-example-h2-ddl.sql can be found in src/test/resources
folder. There is also a service.yml config file to define the DataSource and TodoCommandServer as singletons.

Here is the content of service.yml


```yaml
singletons:
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: org.h2.jdbcx.JdbcDataSource
      jdbcUrl: jdbc:h2:~/test
      username: sa
      password: sa
- com.networknt.eventuate.jdbc.IdGenerator:
  - com.networknt.eventuate.jdbc.IdGeneratorImpl
- com.networknt.tram.todolist.command.TodoCommandService:
  - com.networknt.tram.todolist.command.TodoCommandService

``` 

In the next step, we are going to take a look at [view][] side common commponents.

[view]: /tutorial/tram/todo-list/view/
