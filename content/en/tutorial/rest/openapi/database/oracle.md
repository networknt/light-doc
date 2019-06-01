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
reviewed: true
---

In the previous steps, we have tried the single query, multiple queries, and updates to the MySql database. In this step, we are going to switch from MySql database to Oracle in the configuration service.yml and update pom.xml to include Oracle DB driver as part of the dependency.

The first step is to start the Oracle database in Docker. The command has shown above.

Next, we need to add manually install Oracle JDBC driver jar into your local maven repository. You can search on the Internet on the instructions. If you are not interested in Oracle, please skip this step. The only reason I have Oracle in this tutorial is that one of our customers only have Oracle database as the approved solution. Downloading Oracle XE Docker image will take a long time, and that image is not stable and can be broken at any time. 

To switch to the Oracle database, you need to replace service.yml from dbscript/oracle/config folder. First, let's create a new folder from updates and copy the service.yml

```
cd ~/networknt/light-example-4j/rest/openapi/database
cp -r updates oracle
cp dbscript/oracle/config/service.yml oracle/src/main/resources/config
```

Second, we need to update pom.xml to include the Oracle DB driver. Since Oracle DB driver is not available in maven central, you need to manually download it from oracle.com and manually install it in your local .m2 repository. 

The reason we have include Oracle here is due to some of our customers can only use this database as the corporate policy. If you are part of these type of organizations, then you know how to install the Oracle driver. Otherwise, it is highly recommended to use other open source relational databases like MySql or Postgres. If you are building microservices with separate database instances per service and you have thousands of services, Oracle license cost will be too high to afford even big banks. 

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

Now let's build the server from the oracle folder.

```
cd oracle
mvn clean install exec:exec
```

Now you can test the server with curl to verify that the server is working with Oracle database.

```
curl -k https://localhost:8443/v1/query
```

In the next step, we are going to switch the MySQL database to [Postgres][]. 

[Postgres]: /tutorial/rest/openapi/database/postgres/
