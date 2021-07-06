---
title: "Prepare Environment"
date: 2018-01-06T12:59:09-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In order to work with light-tram-4j, we need to start the infrastructure services and the easier
way is to use docker-compose. Here is a list of services that need to be started before we start
integration testing. 

* Kafka
* Zookeeper
* Mysql
* CDC service for light-tram-4j


To start Kafka, Zookeeper and Mysql, you can use the [docker-compose-eventuate.yml][] in light-docker
repository. And CDC server for light-tram-4j can be found at [docker-compose-cdcserver-for-tram.yml][] 

The detail steps are documented in the [getting started] in light-tram-4j

To review the source code, it would be better to checkout the repository from github and review
the code in an IDE like IntelliJ Idea.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
```

The source code can be found at light-example-4j/tram/light-tram-todolist folder.

There are two different implementations: 

A monolithic single module implementation can be found in single-module folder

A microservices multiple module implementation can be found in multi-module folder.

In the next step, let's walk through the [common][] module which contains domain event definitions.


[docker-compose-eventuate.yml]: https://github.com/networknt/light-docker/blob/master/docker-compose-eventuate.yml
[docker-compose-cdcserver-for-tram.yml]: https://github.com/networknt/light-docker/blob/master/docker-compose-cdcserver-for-tram.yml
[getting started]: /getting-started/light-tram-4j/
[common]: /tutorial/tram/todo-list/common/

