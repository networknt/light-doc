---
title: "Start Services"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 20
aliases: []
toc: false
draft: false
reviewed: true
---

Note: the following steps use Oracle database as an example. MySQL, MariaDB, SQL Server and Postgres should be the same by choosing docker-compose-oauth2-mysql.yml, docker-compose-oauth2-mariadb.yml, docker-compose-oauth2-sqlserver or docker-compose-oauth2-postgres.yml when starting docker-compose.

The current release supports five built-in databases: Mysql, MariaDB, SQLServer, Postgres, and Oracle. If you want to use other databases, you need include the JDBC driver into the classpath and create a datasource config in service.yml file. 

In production mode, all services will have docker images downloaded from hub.docker.com or a private docker hub within your organization. And Kubernetes or other docker orchestration tools will be used to manage containers. 

There are two docker images per service and the Redhat based image is only used by customers with OpenShift Platform as it is the only one that is approved. 

* Alpine Linux based image (78MB) which is smaller and recommended for most users.

* Redhat Linux based image (219MB) with networknt as user instead of root for maximum security. 


To help the user to understand how each service works and enables the user to modify services, the first section of this tutorial will focus on development mode which will build these services and dockerize them. 

For each microservices, we have Dockerfile in the root folder and Dockerfile-Redhat in docker folder. Here is an example:

https://github.com/networknt/light-oauth2/tree/master/client/docker

You can manually build these images one by one per service or leverage [light-bot][] dockerhub task to release all services to docker hub together. For each release of light platform, we will push these service to networknt docker hub. You can start these services with the following commands.


```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-oracle.yml up
```

It will take about 30 seconds to have all services and database up and running. If Oracle XE image doesn't exist on your host, it will take longer to download it.

If you have modified docker container config files or want to clean up, please follow the steps to restart services.
 
```
docker-compose -f docker-compose-oauth2-oracle.yml down
docker-compose -f docker-compose-oauth2-oracle.yml up
```

### Mysql

To start light-oauth2 microservices with Mysql database.

```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up
```

### MariaDB

To start light-oauth2 microservices with MariaDB.

```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-mariadb.yml up
```

### SQLServer

To start light-oauth2 microservices with Microsoft SQL Server.

```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-sqlserver.yml up
```

### Postgres


To start light-oauth2 microservices with Postgres database.

```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-postgres.yml up
```


If you have modified source code, you need to rebuild the updated service and then rebuild the docker image locally. And before rebuild docker image, you need to remove the existing one with: 

```
docker rmi -f [image id]
```


[light-bot]: https://github.com/networknt/light-bot