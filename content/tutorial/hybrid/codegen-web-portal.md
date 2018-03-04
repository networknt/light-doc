---
title: "Codegen Web Portal"
date: 2018-03-04T11:55:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Most developers use [light-codegen][] as a Java command line tool or Docker command line
tool to scaffold their project. There is another option that is still working in progress
which you can utilize a service from light-portal to generate the code from the specification
defined in marketplace. 

There are two parts for the light-codegen in light-portal: an API built with light-hybrid-4j
and a single page application built with React. 

The following tutorial focuses on how to start the light-port/hybrid-query server with code
generator API embedded. 

### Preparation

In order to complete this tutorial, you need to clone the following repository into your
workspace. The following assumes networknt as workspace folder under user home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-config-test.git
git clone https://github.com/networknt/light-portal.git
``` 

The configuration files are located at light-config-test/light-portal/hybrid-query/cloud
folder and you need to go to that folder to start the server. 

First let's build light-codegen so that we can have all the jar files locally. As light-codegen
are released to maven central, you can download the right version of jar files from there as
well. 

```
cd ~/networknt/light-codegen
mvn clean install -DskipTests
``` 

Now let's build the hybrid-query server in light-portal. 

```
cd ~/networknt/light-portal/hybrid-query
mvn clean install
```
 

### Start server with dependencies in pom.xml

By default, hybrid-query project pom.xml include all the dependencies for light-codegen jar
files. The following block can be found in pom.xml file.

```
        <!-- light-codegen dependencies can be removed and put the jar file into service folder. -->
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-core</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-fwk</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-graphql-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-hybrid-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-rest-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-web</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

In this case, the jar files for light-codegen is loaded form maven central when the project
is built. 

To start the server, let's use the externalized config files in light-config-test folder.

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.0.0.jar:../service/* com.networknt.server.Server
```

Once the server is up and running, you can follow the step below Test Codegen Service to
confirm it is working. 

### Start server with dependencies in service folder
 
Putting dependent services into hybrid-query pom.xml file is one way to embed services but 
it is not flexible as you have to rebuild the project in order to change the version of
services. Another way is to add these dependent jar files into a service folder and then 
load dynamically during server startup by putting these files into the classpath. In this
way, if you want to upgrade your services, you just need to copy the new version of jar
files into the service folder and restart the hybrid-query server. 

In summary, it makes sense to include the dependencies into pom.xml during development as
it is very convenient. However, when you goto production, it is wise to put these services
into a separate folder to load them dynamically.

Now let's first copy the jar files to ~/networknt/light-config-test/light-portal/hybrid-query/service
folder from just built light-codegen project. 

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/service
cp ~/networknt/light-codegen/codegen-core/target/codegen-core-1.5.11.jar .
cp ~/networknt/light-codegen/codegen-fwk/target/codegen-fwk-1.5.11.jar .
cp ~/networknt/light-codegen/codegen-web/target/codegen-web-1.5.11.jar .
cp ~/networknt/light-codegen/light-graphql-4j/target/light-graphql-4j-generator-1.5.11.jar .
cp ~/networknt/light-codegen/light-hybrid-4j/target/light-hybrid-4j-generator-1.5.11.jar .
cp ~/networknt/light-codegen/light-rest-4j/target/light-rest-4j-generator-1.5.11.jar .
``` 

Once this is done, you need to comment out the section in pom.xml for all light-codegen
dependencies.

```xml

        <!-- light-codegen dependencies can be removed and put the jar file into service folder. -->
        <!--
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-core</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-fwk</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-graphql-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-hybrid-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>light-rest-4j-generator</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>codegen-web</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
        -->
``` 

Rebuild the hybrid-query server without the light-codegen in the fat jar. 

```
cd ~/networknt/light-portal/hybrid-query
mvn clean install
``` 

Now we can start the server from cloud folder again with light-codegen services loaded
from service folder in classpath. 

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.0.0.jar:../service/* com.networknt.server.Server
```

To confirm the services are working follow the next step to test.


### Test Codegen Service

Given that light-codegen/codegen-web is one of the services, to ensure your hybrid-server
is working, we are going to call one of the action in codegen-web to verify if the server
is functioning.

```
curl -k -X POST \
  https://localhost:8082/api/json \
  -H 'cache-control: no-cache' \
  -d '{"host":"lightapi.net","service":"codegen","action":"listFramework","version":"0.0.1"}'
```

and the expected result should be

```
["light-hybrid-4j-server","light-graphql-4j","openapi","light-hybrid-4j-service","swagger"]
```


[light-codegen]: /tool/light-codegen/
