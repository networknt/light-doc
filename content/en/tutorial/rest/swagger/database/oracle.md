---
title: "Oracle"
date: 2017-11-24T16:18:39-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In previsou steps, we have tried single query, multiple queries and updates to
Mysql database. In this step, we are going to switch from Mysql database to Oracle
in the configuration service.yml and update pom.xml to include Oracle db driver
as part of the dependency.

The first step is to start Oracle database in Docker. The command has shown above.

Next we need to add manually install Oracle JDBC driver jar into your local maven
repository. You can search on Internet on the instructions. If you are not interested 
in Oracle, please just skip this step. The only reason I have Oracle in this tutorial
is because one of our customers only have Oracle database as approved. Downloading
Oracle XE Docker image will take a long time and that image is not stable and can be
broken at anytime. 

To switch to Oracle database, you just need to replace service.yml from
dbscript/oracle/config folder. First let's create a new folder from
updates and copy the service.yml

```
cd ~/networknt/light-example-4j/rest/swagger/database
cp -r updates oracle
cp dbscript/oracle/config/service.yml oracle/src/main/resources/config
```

Second, we need to update pom.xml to include Oracle db driver. Since Oracle db driver
are not available in maven central, you need to manually download it from oracle.com
and manually install it in your local .m2 repository. 

The reason we have include Oracle here is due to some of our customers can only this
this database as the corporate policy. If you are part of these type of organizations,
then you know how to install Oracle driver. Otherwise, it is highly recommended to
use other open source relational databases like Mysql or Postgres. If you are building
microservices with separate database instances per service and you have thousands of
services, Oracle license cost will be too high to afford even big banks. 

Here is a line that needs to be added into pom.xml <properties></properties> section

```
        <version.oracle>11.2.0.3</version.oracle>

```


Here are several lines to be added into the dependencies section.

```
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>${version.oracle}</version>
        </dependency>

```  

Now let's build the server from oracle folder.

```
cd oracle
mvn clean install exec:exec
```

Now you can test the server with curl to verify that the server is working with 
Oracle database.

```
curl -k https://localhost:8443/v1/query
```

