---
title: "Host Menu Command Portal"
date: 2018-03-04T17:28:17-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This tutorial shows you how to load host-menu command side hybrid service with the hybrid-command
server in light-portal.

The host-menu is a service that you can define menu and menu item for a web site on light-portal.
It can also load a hierarchical menu for a particular host. 

The command side service is built with light-hybrid-4j and light-eventuate-4j and it is responsible
for add/update/delete menu and menu item.

### Build

First let's build host-menu locally. 

```
cd ~/networknt/light-portal/host-menu
mvn clean install

``` 

Now let's copy the jar files to the service folder for the hybrid-command. 

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/service
cp ~/networknt/light-portal/host-menu/common/target/host-menu-common-0.1.0.jar .
cp ~/networknt/light-portal/host-menu/command/target/host-menu-command-0.1.0.jar .
cp ~/networknt/light-portal/host-menu/hybrid-command/target/menu-command-0.1.0.jar .
``` 

Before starting the server, we need to ensure that eventuate dependencies are included into
pom.xml file for hybrid-command project. 

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

After update pom.xml, the hybrid-command project in light-portal needs to be rebuilt. 

```
cd ~/networknt/light-portal/hybrid-command
mvn clean install
```

Let's start the server with the following command.

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-command/target/hybrid-command-1.0.0.jar:../service/* com.networknt.server.Server
```

### Test

Let's create a new menu for a sample host. 

```
curl -k -H "Content-Type: application/json" -X POST -d '{"host":"lightapi.net","service":"menu","action":"createMenu","version":"0.1.0","data":{"host":"example.org","description":"example org web site","contains":["1","2","3"]}}' https://localhost:8081/api/json
```

And the result is.

```json
{"host":"example.org","description":"example org web site","contains":["1","2","3"]}
```
