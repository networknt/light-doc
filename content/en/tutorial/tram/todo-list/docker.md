---
title: "Docker"
date: 2018-01-08T19:14:18-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have started both command side and query side with Java
command line or IDE. Here we are going to build document images for both services
and start them all together with ElasticSearch in a docker-compose. 


The docker-compose.yml can be found in the root directory and inside, there are 
three services including ElasticSearch. 

The config files for these two services will be changed for service.yml and kafka.yml
so we have externalized these into docker folder. 

Let's first use Ctrl+C to shutdown the single ElasticSearch docker-compose and two
services started with java -jar.

 
### Start servers

Let's start the server with docker-compose.yml in the root directory.

```
docker compose up
```


### Test

Once servers are up and running, we can issue the same command to access the command side
service. 

```
curl -X POST   http://localhost:8081/v1/todos   -H 'Cache-Control: no-cache'   -H 'Content-Type: application/json'   -d '{"title":"canada","completed":false,"order":0}'
```

And to access the query or view side.

```
curl -X GET   'http://localhost:8082/v1/todoviews?searchValue=canada'   -H 'Cache-Control: no-cache'   -H 'Content-Type: application/json'
```

### Summary

This tutorial shows up another way to implement CQRS without event sourcing as in [light-eventuate-4j][]
framework. It is much simpler without dealing with event store; however, it doesn't have the benefits of
event sourcing. If you are interested in event sourcing implementation, please refer to [todo-list][].



[light-eventuate-4j]: /style/light-eventuate-4j/
[todo-list]: /tutorial/eventuate/todo-list/
