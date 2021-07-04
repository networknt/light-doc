---
title: "Schema Form Query Portal"
date: 2018-03-05T06:47:09-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This tutorial shows how to load schema form query side service from hybrid-query in 
light-portal. 

After [host-menu query side][] integration, it should be very straightforward to add
another light-eventuate-4j service to the hybrid-query server. 

### Build

First the schema-form in light-portal needs to be built.

```
cd ~/networknt/light-portal/schema-form
mvn clean install
```

It is easier to add the common and query of schema form to the pom.xml in hybrid-query
pom.xml file than copy to service folder. 

```xml
        <!-- schema form query side dependencies -->
        <dependency>
            <groupId>net.lightapi</groupId>
            <artifactId>schema-form-common</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>net.lightapi</groupId>
            <artifactId>schema-form-query</artifactId>
            <version>0.1.0</version>
        </dependency>

```

After update pom.xml, the hybrid-query project in light-portal needs to be rebuilt. 

```
cd ~/networknt/light-portal/hybrid-query
mvn clean install
```


Now let's copy the schema query service to the service folder for the hybrid-query in
light-config-test.

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/service
cp ~/networknt/light-portal/schema-form/hybrid-query/target/form-query-0.1.0.jar .
``` 

Use the following command to start the server.

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.0.0.jar:../service/* com.networknt.server.Server
```



### Test

Tobe completed.

[host-menu query side]: /tutorial/hybrid/host-menu-query-portal/
