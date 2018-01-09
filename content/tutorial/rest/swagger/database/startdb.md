---
title: "Startdb"
date: 2017-11-24T11:23:03-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 60	#rem
aliases: []
toc: false
draft: false
---

In the previous step, we've already prepared the scripts for Oracle, Mysql and Postgres
databases and put these script file under light-example-4j/rest/swagger/database. At
this moment, you should have two sub-folders under database. generated and dbscript.


In order to work on our service, we need to start database standalone for now. Depending
on which database you are working on, you can choose one of them below. For this demo 
use mysql and later on we can switch to Postgres and Oracle.


Oracle Database

```
docker run -v ~/networknt/light-example-4j/rest/swagger/database/dbscript/oracle:/docker-entrypoint-initdb.d -d -p 1522:1521 wnameless/oracle-xe-11g
```

 
Mysql Database

```
docker run -v ~/networknt/light-example-4j/rest/swagger/database/dbscript/mysql:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d -p 3306:3306 mysql

```

Postgres Database

```
docker run -v ~/networknt/light-example-4j/rest/swagger/database/dbscript/postgres:/docker-entrypoint-initdb.d -e POSTGRES_PASSWORD=my-secret-pw -e POSTGRES_DB=hello_world -d -p 5432:5432 postgres

```

In this step, let's first start the Mysql database and in the next step, we will setup the
[database connection pool][] in our project.

 
[database connection pool]: /tutorial/rest/swagger/database/connection-pool/
 
