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

Note: the following steps use MySQL database as an example. MariaDB, SQL Server and Postgres should be the same by choosing docker-compose-oauth2-mariadb.yml, docker-compose-oauth2-sqlserver.yml or docker-compose-oauth2-postgres.yml when starting docker-compose.

The current release supports four built-in databases: MySQL, MariaDB, SQLServer and Postgres. If you want to use other databases, you need include the JDBC driver into the classpath and create a datasource config in service.yml file. 

In production mode, all services will have docker images downloaded from hub.docker.com or a private docker hub within your organization. And Kubernetes or other docker orchestration tools will be used to manage containers. 

For Java 8 in 1.6.x branch, there are two docker images per service and the Redhat based image is only used by customers with OpenShift Platform as it is the only one that is approved. 

* Alpine Linux based image (117MB) which is smaller and recommended for most users.

* Redhat Linux based image (485MB) with networknt as user instead of root for maximum security. 

For Java 11 in master branch, there are two docker images per service. 

* Alpine Linux based with jlink image (95MB) which is recommended for most users. 
* Slim Linux based image (422MB) for user who want to have a full Linux container. 

To help the user to understand how each service works and enables the user to modify services, the first section of this tutorial will focus on development mode which will build these services and dockerize them. 

For each microservices, we have Dockerfile in the root folder and Dockerfile-Redhat in docker folder. Here is an example:

https://github.com/networknt/light-oauth2/tree/master/client/docker

You can manually build these images one by one per service or leverage [light-bot][] dockerhub task to release all services to docker hub together. For each release of Light, we will push these service to networknt docker hub. You can start these services with the following commands.


```
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up
```

It will take about 30 seconds to have all services and database up and running. If MySQL image doesn't exist on your host, it will take longer to download it.

If you have modified docker container config files or want to clean up, please follow the steps to restart services.
 
```
docker-compose -f docker-compose-oauth2-mysql.yml down
docker-compose -f docker-compose-oauth2-mysql.yml up
```

If this is the very first time you start the docker-compose, you might encounter an error as below. 

```
ERROR: Network localnet declared as external, but could not be found. Please create the network manually using `docker network create localnet` and try again.
```

It is because we are using a docker network called `localnet` in the docker-compose. As the error message indicates, we need to run the following command to create the network and then rerun the docker-compose. 

```
docker network create localnet
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

### Deprecated Databases

We used to support Oracle and MariaDB but marked them deprecated due to some issues. 

Oracle was removed due to the licensing issue for the Oracle Client. The Oracle Client driver is not in the maven central and needs a registered user to access from the Oracle repository. Also, the docker image is too big to download. For anyone to use Oracle, they might have Oracle installed already, and it is not necessary to demo with docker here. 

MariaDB shares the same driver with MySQL, and MySQL supports SSL by default. We have updated the driver configuration to support SSL, but there is no way for us to enable SSL with the MariaDB docker container. So we have to mark the docker-compose deprecated.  

Although the above databases are not supported with docker-compose, you can still use it with a small update in the application. The database scripts are still available for users. 


[light-bot]: https://github.com/networknt/light-bot