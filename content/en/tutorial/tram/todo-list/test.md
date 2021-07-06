---
title: "Build and Test"
date: 2018-01-08T10:04:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

By now, we have walked through each sub-modules. Let's build and test the services
we have built. 

### Build

Here are the commands to build the project and run the test cases.

```
cd ~/networknt/light-example-4j/tram/light-tram-todolist/multi-module
mvn clean install
```

### Start

There are two different ways to start servers. Start them as a standalone servers with
java -jar or start them with docker-compose all together. 

If you are still in development and want to test the code or debug individual service
start server with java -jar or within IDE would make more sense. 

As our view service is depending on ElasticSearch, let's first start it in a docker-compose.


```
docker-compose -f docker-compose-elasticsearch.yml up
```

Two things should be verified before running the service:

1. since we are using Elasticsearch for view persistence, please increase the docker memory 
space to 4gb  for the docker. The problem seems to be only on Mac Book Pro and Linux is fine.

    -- please refer issue: https://github.com/10up/wp-local-docker/issues/6

2. current default Elasticsearch host setting is for Mac, if you running the service for Linux, 
change it to the network IP like "192.168.1.120" or the IP address running Elasticsearch

   - com.networknt.tram.todolist.view.TodoViewService:
     - com.networknt.tram.todolist.view.TodoViewServiceImpl:
       - java.lang.String: 10.200.10.1

For more information about DOCKER_HOST_IP, please refer to [eventuate getting started][]

For me to run the application on Linux, I have to change the above IP from 10.200.10.1 to
192.168.1.120 in service.yml file under tram-todo-view/src/main/resources/config


Let's start the command service first

```
cd ~/networknt/light-example-4j/tram/light-tram-todolist/multi-module/tram-todo-command
java -jar target/tram-todo-command-service-1.0.0.jar

```

Next let's start view service.

```
cd ~/networknt/light-example-4j/tram/light-tram-todolist/multi-module/tram-todo-view
java -jar target/tram-todo-view-service-1.0.0.jar
```

### Test

It is very easy to test with curl command as both sides are implemented as restful services.

To create a new TODO item, 

```
curl -X POST   http://localhost:8081/v1/todos   -H 'Cache-Control: no-cache'   -H 'Content-Type: application/json'   -d '{"title":"canada","completed":false,"order":0}'
```

And you might have a response like this.

```
{"id":"00000160d83cd066-8eb59f5f03eb0000","title":"canada","completed":false,"executionOrder":0}
```

Now let's access the query or view side.

```
curl -X GET   'http://localhost:8082/v1/todoviews?searchValue=canada'   -H 'Cache-Control: no-cache'   -H 'Content-Type: application/json'
```

And the response will be something like this.

```
[{"id":"00000160d83cd066-8eb59f5f03eb0000","title":"canada","completed":false,"executionOrder":0},{"id":"00000160d8408c1f-8eb59f5f03eb0000","title":"canada","completed":false,"executionOrder":0}]
```

In the next step, we are going to use [Docker][] for all services. 


[eventuate getting started]: /tutorial/eventuate/getting-started/
[Docker]: /tutorial/tram/todo-list/docker/
