---
title: "CDC server vs service"
date: 2018-01-06T19:34:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In light-eventuate-4j we have two different CDC implementations mysql binlog and polling. The first only works with MySql database and it is the most efficient way. The second polling is a generic implementation that is suitable for any SQL database like Postgres or Oracle. 

For each implementation, we need to start a server to allocate a thread to do the CDC work. There are two different ways to implement the server. light-rest-4j or light-hybrid-4j. 

### eventuate-cdc-server

This is a microservice implemented on top of light-rest-4j framework. It constantly
monitors Mysql binlog and pushes EVENTS table changes to Kafka topic. This server can be
started with a docker-compose file in [light-docker](https://github.com/networknt/light-docker)

Assume you are using ~/networknt as your working directory.

```
git clone https://github.com/networknt/light-docker.git
docker-compose -f docker-compose-cdcserver.yml up
```


### eventuate-cdc-service

This is a microservice implemented on top of light-hybrid-4j framework. It monitors Mysql
binlog and pushes EVENTS table changes to Kafka topic. Unlike the cdc-server above, this
service cannot be started standalone. It must be embedded into hybrid-command server
described below.

Note: the entire light-eventuate-4j platform only need one Data Change Capture service and
you can choose the restful cdc-server or hybrid cdc-service but make sure that only one of
them is running.


