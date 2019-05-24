---
title: "Schema Form Command Portal"
date: 2018-03-05T06:46:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Once the [Host Menu Command Side][] is completed, it is very easy to add another eventuate
based command service to the hybrid-command server. 

schema-form services are used to management forms in light-portal so that all UI can be
defined in [React schema form][] and generated automatically. 

### Build

First the schema-form in light-portal needs to be built.

```
cd ~/networknt/light-portal/schema-form
mvn clean install
```

It is easier to add the common and command of schema form to the pom.xml in hybrid-command
pom.xml file than copy to service folder. 

```xml
        <!-- schema form common and command -->
        <dependency>
            <groupId>net.lightapi</groupId>
            <artifactId>schema-form-common</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>net.lightapi</groupId>
            <artifactId>schema-form-command</artifactId>
            <version>0.1.0</version>
        </dependency>

```

After update pom.xml, the hybrid-command project in light-portal needs to be rebuilt. 

```
cd ~/networknt/light-portal/hybrid-command
mvn clean install
```


Now let's copy the schema command service to the service folder for the hybrid-command in
light-config-test.

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/service
cp ~/networknt/light-portal/schema-form/hybrid-command/target/form-command-0.1.0.jar .
``` 

Use the following command to start the server.

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-command/target/hybrid-command-1.0.0.jar:../service/* com.networknt.server.Server
```


### Test

Tobe completed.


[Host Menu Command Side]: /tutorial/hybrid/host-menu-command-portal/
[React schema form]: https://github.com/networknt/react-schema-form
 