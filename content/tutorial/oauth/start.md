---
title: "Start"
date: 2017-11-10T14:47:50-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Note: the following steps use Oracle database as an example. MySQL and Postgres should be the same
by choosing docker-compose-oauth2-mysql.yml or docker-compose-oauth2-postgres.yml when starting docker-compose.


In production mode, all services will have docker images downloaded from hub.docker.com or private
docker hub within your organization. And Kubernetes or other docker orchestration tools will be
used to manage containers. 

To help user to understand how each service work and enable user to modify services, the first section
of this tutorial will focus on development mode which will build these services and dockerize them. 

The following will check out the repo, build and start services with Oracle XE database. Please don't
use this in your production as Oracle XE is not supposed to be used in production. If you want to use
the light-oauth2 in production, you can use Mysql or Postgres.    

```
git clone git@github.com:networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-oauth2-oracle.yml up
```

It will take about 30 seconds to have all services and database up and running. If Oracle XE image
doesn't exist on your host, it will take longer to download it.

If you have modified source code, please follow the steps to restart services.
 
```
docker-compose -f docker-compose-oauth2-oracle.yml down
docker-compose -f docker-compose-oauth2-oracle.yml up
```


