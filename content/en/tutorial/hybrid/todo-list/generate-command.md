---
title: "Generate Command Service"
date: 2019-04-09T10:56:14-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have the [workspace parepared][] and the light-codegen is built lcally. In this step, we are going to generate the command service.

As we know, to generate a hybrid service, we need to have a config.json and a schema.json as inputs for the light-codegen. For this tutorial, we have the command service config and schema created in the model-config repository already. They are located at https://github.com/networknt/model-config/tree/master/hybrid/todo-command

config.json

```
{
  "rootPackage": "com.networknt.todolist.command",
  "handlerPackage":"com.networknt.todolist.command.handler",
  "modelPackage":"com.networknt.todolist.command.model",
  "artifactId": "hybrid-command",
  "groupId": "com.networknt",
  "name": "hybrid-command",
  "version": "0.1.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": false,
  "supportPostgresql": false,
  "supportH2ForTest": false,
  "supportClient": false,
  "dockerOrganization": "networknt"
}
```

schema.json

```
{
  "host": "lightapi.net",
  "service": "todo",
  "action": [
   {
    "name": "delete",
    "version": "0.1.0",
    "handler": "DeleteTodo",
    "scope" : "todo.w",    
    "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "id" : {
          "type" : "string"
        }
      },
      "required" : [ "id"]
    },
    "scope" : "todo.w"
  },
  {
      "name": "create",
      "version": "0.1.0",
      "handler": "CreateTodo",
      "scope" : "todo.w",
      "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "title" : {
          "type" : "string"
        },
        "completed" : {
          "type" : "boolean"
        },
        "order" : {
          "description" : "order of todo in the todo list",
          "type" : "integer",
          "minimum" : 0
        }
      },
      "required" : [ "title" ]
    },
    "scope" : "todo.w"
  },
{
      "name": "update",
      "version": "0.1.0",
      "handler": "UpdateTodo",
      "scope" : "todo.w",
      "schema" : {
      "title" : "Service",
      "type" : "object",
      "properties" : {
        "id" : {
          "type" : "string"
        },
        "title" : {
          "type" : "string"
        },
        "completed" : {
          "type" : "boolean"
        },
        "order" : {
          "description" : "order of todo in the todo list",
          "type" : "integer",
          "minimum" : 0
        }
      },
      "required" : [ "id" ]
    },
    "scope" : "todo.w"
  }
  ]
}

```

Run the following commands to generate the command service into the light-example-4j/eventuate/todo-list folder. 

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/eventuate/todo-list/hybrid-command -c model-config/hybrid/todo-command/config.json -m model-config/hybrid/todo-command/schema.json
```

Once the service is generated, we need to update it. 

pom.xml

First we need to add parent section to ensure that this service moduel is part of the todo-list application in the eventuate. 

```
    <parent>
        <groupId>com.networknt</groupId>
        <artifactId>light-eventuate-todo</artifactId>
        <version>0.1.0</version>
        <relativePath>..</relativePath>
    </parent>

```


[workspace parepared]: /tutorial/hybrid/todo-list/workspace/
