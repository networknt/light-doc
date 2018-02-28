---
title: "Hybrid Services Test"
date: 2018-01-07T16:44:33-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


We will start hybrid-command server and hybrid-query server manually first and
then start them together along with services in a docker-compose.

The first step is to make sure light-eventuate-4j platform is up and running.
Please follow this [tutorial][]

To start hybrid servers manually, we need to checkout light-eventuate-4j as these
two servers are part of it.

```
cd ~/networknt
git clone git@github.com:networknt/light-eventuate-4j.git
cd light-eventuate-4j
mvn clean install
```

Next let's start hybrid-command server with cdc-service and todo-list hybrid-command
service. 

As there are more jars that need to be on the classpath of hybrid-command server
we are going to copy all of them into light-docker/

```
cd ~/networknt/light-docker/eventuate/hybrid-command/service

cp ~/networknt/light-example-4j/eventuate/todo-list/common/target/todo-common-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/command/target/todo-command-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/hybrid-command/target/hybrid-command-0.1.0.jar .
```

```
cd ~/networknt/light-eventuate-4j/hybrid-command

mvn clean install

java -cp ~/networknt/light-docker/eventuate/hybrid-command/service/*:target/hybrid-command-1.0.0.jar com.networknt.server.Server

```


Next let's start hybrid-query server with todo-list hybrid-query
service. 

As there are more jars that need to be on the classpath of hybrid-query server
we are going to copy all of them into light-docker/

```
cd ~/networknt/light-docker/eventuate/hybrid-query/service
cp ~/networknt/light-example-4j/eventuate/todo-list/common/target/todo-common-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/command/target/todo-command-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/query/target/todo-query-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/hybrid-query/target/hybrid-query-0.1.0.jar .
```

```
cd ~/networknt/light-eventuate-4j/hybrid-query

mvn clean install

java -cp ~/networknt/light-docker/eventuate/hybrid-query/service/*:target/hybrid-query-1.0.0.jar com.networknt.server.Server

```

Now let's create a todo item on command service with the following command.

```
curl -X POST \
  http://localhost:8083/api/json \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{"host":"lightapi.net","service":"todo","action":"create", "version": "0.1.0", "title": "create todo item from hybrid-command", "completed": false, "order": 1}'
```

Let's get all the todo items from query service with the following command

```
curl -X POST \
  http://localhost:8082/api/json \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{"host":"lightapi.net","service":"todo","action":"gettodos", "version": "0.1.0"}'
```

[tutorial]: /tutorial/eventuate/getting-started/
