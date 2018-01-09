---
title: "Hybrid Docker"
date: 2018-01-07T16:45:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


Copy service jar files to service folder:

For hybrid command service :

```
cd ~/networknt/light-docker/eventuate/hybrid-command/service

cp ~/networknt/light-example-4j/eventuate/todo-list/common/target/todo-common-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/command/target/todo-command-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/hybrid-command/target/hybrid-command-0.1.0.jar .
cp ~/networknt/light-eventuate-4j/hybrid-command/target/hybrid-command-1.0.0.jar .
```

For hybrid query:

```
cd ~/networknt/light-docker/eventuate/hybrid-query/service
cp ~/networknt/light-example-4j/eventuate/todo-list/common/target/todo-common-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/command/target/todo-command-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/query/target/todo-query-0.1.0.jar .
cp ~/networknt/light-example-4j/eventuate/todo-list/hybrid-query/target/hybrid-query-0.1.0.jar .
cp ~/networknt/light-eventuate-4j/hybrid-query/target/hybrid-query-1.0.0.jar .

```

Go to light-docker root folder and start docker-compose file for eventutate hybrid service:

```
cd ~/networknt/light-docker/

docker-compose -f docker-compose-eventuate-hybrid-local.yml up

```

Verify result with following command

Create a todo item on command service with the following command.

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

