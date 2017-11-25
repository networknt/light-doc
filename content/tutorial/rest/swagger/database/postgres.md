---
title: "Postgres"
date: 2017-11-24T16:18:33-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In previsou steps, we have switched to Oracle database from Mysql and in this step,
we are going to switch from Oracle to Postgres to further demonstrate how to config
different database without change the service code. The only thing that need to update
is the configuration service.yml and pom.xml to include Postgres db driver as part of 
the dependency.

The first step is to start the postgres database in docker. The command
has shown in previous step [start databases][] 


To switch to Postgres database, you just need to replace service.yml from 
dbscript/postgres/config folder. First let's create a new folder from 
updates and copy the service.yml

```
cd ~/networknt/light-example-4j/rest/swagger/database
cp -r updates postgres
cp dbscript/postgres/config/service.yml postgres/src/main/resources/config
```

Second let's add Postgres db driver to the pom.xml

In the properties section, add the following line. 

```
        <version.postgres>42.1.1</version.postgres>        

```

In the dependencies section, add the following lines.

```
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>${version.postgres}</version>
        </dependency>

```


Now let's build the server from postgres folder.

```
cd postgres
mvn clean install exec:exec
```

Now you can test the server with curl to verify that the server is working with 
Postgres database.

```
curl -k https://localhost:8443/v1/query
```

[start databases]: /tutorial/rest/swagger/database/startdb/
