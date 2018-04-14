---
title: "Rest Services Test"
date: 2018-01-07T16:42:19-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Once all modules can be built successfully, we can start the servers and test the todo-list application.

First, we need to make sure Mysql, Zookeeper, Kafka and CDC server for eventuate are up and running.

You can follow this [tutorial][] to start all of them with a docker-compose.

Before we start the rest-command and rest-query, let's create the database and table for the rest-query material view in the same Mysql database for the event store. We are going to create another database called todo_db. If you are using the networknt/mysql docker image, the table is created already. 

Here is the database script and you can use mysql command line or just using any GUI tools to run it against MySQL server.

```mysql
create database todo_db;

GRANT ALL PRIVILEGES ON todo_db.* TO 'mysqluser' IDENTIFIED BY 'mysqlpw';

use todo_db;

DROP table IF EXISTS  TODO;


CREATE  TABLE TODO (
  ID varchar(255),
  TITLE varchar(255),
  COMPLETED BOOLEAN,
  ORDER_ID INTEGER,
  ACTIVE_FLG varchar(1) DEFAULT 'Y',
  PRIMARY KEY(ID)
);

```

Remember that MySQL root user and password as follows.

```
dbUser: root
dbPass: rootpassword

```

Now let's start rest-command service

```
cd ~/networknt/light-example-4j/eventuate/todo-list/rest-command
java -jar target/rest-command-1.0.0.jar
```

Now let's start rest-query service

```
cd ~/networknt/light-example-4j/eventuate/todo-list/rest-query
java -jar target/rest-query-1.0.0.jar
```


Let's create a todo item with curl

```
curl -k -X POST \
  https://localhost:8081/v1/todos \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{"title":"this is the test todo from postman","completed":false,"order":0}'
```

And the response will be 

```json
{
    "cancelled": false,
    "done": true,
    "completedExceptionally": false,
    "numberOfDependents": 0
}
```

This request will send a request which will call back-end service to generate a "create todo" event and publish to event store.

Event sourcing system will save the event into event store.

CDC service will be triggered and will publish the event to Kafka:

The request publishes a "CreateTodo" event and will save the entity/event to the event store MySQL database.

We can use mysql command line tool to verify:

```
select * from entity;

select * from events;
```


Now let's access the rest-query service

```
curl -k -X GET https://localhost:8082/v1/todos
```

And the response will be something like this.

```json
[
    {
        "0000015d968eacee-0e23a9398a990001": {
            "title": "this is the test todo from postman",
            "completed": false,
            "order": 0
        }
    },
    {
        "0000015d968f5443-0e23a9398a990001": {
            "title": "this is the test todo from postman",
            "completed": false,
            "order": 0
        }
    },
    {
        "0000015d969d6ce2-0e23a9398a990001": {
            "title": "this is the test todo from postman",
            "completed": false,
            "order": 0
        }
    }
]
```

Event sourcing system will subscribe the events from event store and process the events by user-defined event handlers.

For todo-list example, the event handler simply gets the event and save the latest todo info into local TODO table.


In the next step, we are going to implement a [hybrid command][] side with the light-hybrid-4j framework. 

[tutorial]: /tutorial/eventuate/getting-started/
[hybrid command]: /tutorial/eventuate/todo-list/hybrid-command/
