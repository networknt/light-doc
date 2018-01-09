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
---


Let's copy docker config folder to the todo-list folder from existing
one.

- docker  contains docker config files


```
cd ~/networknt/light-example-4j/eventuate/todo-list
cp -r ../todo-list.bak/docker .

```

There are two sub-folders in the docker module:


\command-service      -- command side config, include service.yml file

Since the service runs in the docker container, we need change the mysql host name to use docker image name:

service.yml:

```yaml
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://mysql:3306/eventuate?useSSL=false
      username: mysqluser
      password: mysqlpw

```

\query-service        -- query side config, include service.yml and kafka.yml file

Since the service runs in the docker container, we need change the mysql host name and kafka host name to use docker image name:

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

## Start command side and query side service from docker compose

Instead of start service from command line, start service from docker compose file:

```
cd ~/networknt/light-example-4j/eventuate/todo-list
docker-compose up

```

Then following same steps above to verify the service result from command line


