---
title: "Rest Docker"
date: 2018-01-07T16:45:31-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Let's copy docker config folder to the todo-list folder from the existing one.


```
cd ~/networknt/light-example-4j/eventuate/todo-list
cp -r ../todo-list.bak/docker .

```

There are two sub-folders in the docker module:

### command-service

Since the service runs in the docker container, we need to change the MySQL hostname to use docker image name:

service.yml:

```yaml
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://mysql:3306/eventuate?useSSL=false
      username: mysqluser
      password: mysqlpw

```

### query-service

Since the service runs in the docker container, we need to change the MySQL hostname and Kafka hostname to use docker image name:

service.yml:

```yaml
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://mysql:3306/todo_db?useSSL=false
      username: mysqluser
      password: mysqlpw

```

kafka.yml:

```yaml
bootstrapServers: kafka:9092

```

### Start command side and query side service from docker compose

Instead of start service from the command line, start service from docker compose file:

```
cd ~/networknt/light-example-4j/eventuate/todo-list
docker-compose up

```

Now we can follow the same steps in [rest test][] to verify the service result from with curl commands.

In the next step, we are going to [dockerize the hybrid services][]. 

[rest test]: /tutorial/eventuate/todo-list/rest-test/
[dockerize the hybrid services]: /tutorial/eventuate/todo-list/hybrid-docker/



