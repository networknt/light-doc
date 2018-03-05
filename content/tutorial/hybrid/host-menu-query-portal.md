---
title: "Host Menu Query Portal"
date: 2018-03-04T17:28:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This tutorial shows you how to load host-menu query side hybrid service with the hybrid-query
server in light-portal.

The host-menu is a service that you can define menu and menu item for a web site on light-portal.
It can also load a hierarchical menu for a particular host. 

The query side service is built with light-hybrid-4j and light-eventuate-4j and it is responsible
for query menu and menu item.

### Build

First let's build host-menu locally. 

```
cd ~/networknt/light-portal/host-menu
mvn clean install

``` 

The host-menu query side service will need ArangoConfig defined in light-portal/common-util.
Let's build that project.

```
cd ~/networknt/light-portal/common-util
mvn clean install
```

Now let's copy the jar files to the service folder for the hybrid-command. 

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/service
cp ~/networknt/light-portal/host-menu/common/target/host-menu-common-0.1.0.jar .
cp ~/networknt/light-portal/host-menu/query/target/host-menu-query-0.1.0.jar .
cp ~/networknt/light-portal/host-menu/hybrid-query/target/menu-query-0.1.0.jar .
cp ~/networknt/light-portal/common-util/target/common-util-0.1.0.jar .
``` 

Before starting the server, we need to ensure that eventuate dependencies are included into
pom.xml file for hybrid-query project. 

```xml
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-common</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-client</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-server-jdbc</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>eventuate-client-jdbc-common</artifactId>
            <version>${version.light-eventuate-4j}</version>
        </dependency>

```

After update pom.xml, the hybrid-query project in light-portal needs to be rebuilt. 

```
cd ~/networknt/light-portal/hybrid-query
mvn clean install
```

Let's start the server with the following command.

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.0.0.jar:../service/* com.networknt.server.Server
```

### Test

Let's query a menu for a sample host. 

```
curl -k -H "Content-Type: application/json" -X POST -d '{"host":"lightapi.net","service":"menu","action":"getMenuByHost","version":"0.1.0","data":{"host":"example.com"}}' https://localhost:8082/api/json
```


And the result is.

```json
{"contains":[{"contains":[{"route":"/admin/ruleAdmin","roles":["ruleAdmin","admin","owner"],"_rev":"_WdLhKSy--_","entityId":"e12","_id":"menuItem/12","label":"Rule Admin","_key":"12"},{"route":"/admin/formAdmin","roles":["formAdmin","admin","owner"],"_rev":"_WdLhKV6--_","entityId":"e13","_id":"menuItem/13","label":"Form Admin","_key":"13"},{"route":"/admin/hostAdmin","roles":["admin","owner"],"_rev":"_WdLhKQy--_","entityId":"e11","_id":"menuItem/11","label":"Host Admin","_key":"11"}],"route":"/admin","roles":["admin","owner"],"_rev":"_WdLhKX2--_","entityId":"e1","_id":"menuItem/1","label":"admin","_key":"1"},{"route":"/login","roles":["user"],"_rev":"_WdLhKha--_","entityId":"e2","_id":"menuItem/2","label":"login","_key":"2"},{"route":"/logout","roles":["user"],"_rev":"_WdLhKjW--_","entityId":"e3","_id":"menuItem/3","label":"logout","_key":"3"}],"_rev":"_WdLhKlO--_","description":"example site","entityId":"e1","_id":"menu/example.com","_key":"example.com"}
```
