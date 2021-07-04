---
title: "Hybrid Server"
date: 2019-01-05T12:19:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Hybrid server is a special project that is designed as a host server for services built on top
of light-hybrid-4j framework. Every service is built as a small jar file with only the business
logic handlers inside. Multiple jar files can be put into a docker volume to be loaded by a
hybrid server instance running in docker container. In this way, the developer has nothing to 
do with the server at all they just need to focus on their domain business logic. Also as all
these services are deployed in the same JVM, the communication between them are faster although
it is still through HTTPS channel. This is our serverless solution.   

## Input

#### Model

This generator only generates the server platform that can host many other services. It doesn't 
have any model for input.

#### Config

Here is an example of config.json for light-hybrid-4j server generator.
 
```json
{
  "name": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "httpPort": 8080,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false
}
```

| **Field Name** | **Description** |
|------------------|:---------------------------------------------------------------------------------------------------------------------:|
| **name** | used in generated pom.xml for project version |
| **version** | used in generated pom.xml for project version |
| **groupID** | used in generated pom.xml for project groupId |
| **artifactId** | used in generated pom.xml for project artifactId |
| **rootPackage** | the root package name for your project and it will normally be your domain plug project name. |
| **handlerPackage** | the Java package for all generated handlers. |
| **modelPackage** | the Java package for all generated models or POJOs |
| **httpPort** | the port number of Http listener if enableHttp is true. |
| **enableHttp** | to specify if the server listens to http port. Http should only be enabled in dev. |
| **httpsPort** | the port number of Https listener if enableHttps is true. |
| **enableHttps** | to specify if the server listens to https port. Https should be used in any official environment for security reason. |
| **enableRegistry** | to control if built-in service registry/discovery is used. Only necessary if running as standalone java -jar xxx. |
| **supportDb** | to control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml |
| **dbInfo** | section is the database connection pool configuration info. |
| **supportH2ForTest** | if true, add H2 in pom.xml as test scope to support unit test with H2 database. |
| **supportClient** | if true, add com.networknt.client module to pom.xml to support service to service call. |


## Usage

#### Java Command line

You can [download or build][] the codegen-cli command line jar. 

The generator scaffolds a server platform which can host a number of light-hybrid-4j services.
The generated project is just a skeleton without any service. You have to generate one or more 
services (by using the light-hybrid-4j service generator) and put these jar file(s) into the 
classpath and start this server.

For more information about light-hybrid-4j, please refer to the [project][] 
and its [documentation][].

The following will generate a sample light-hybrid-4j **server** based on test configuration for 
a petstore. 

Working directory: light-codegen

```
java -jar codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-server -o /tmp/hybridserver -c light-hybrid-4j/src/test/resources/serverConfig.json
```


#### Docker Command Line

The following command is using docker to generate light-hybrid-4j server into 
/tmp/light-codegen/hybridserver folder

```
docker run -it -v ~/networknt/light-codegen/light-hybrid-4j/src/test/resources:/light-api/input -v /tmp/light-codegen:/light-api/out networknt/light-codegen -f light-hybrid-4j-server -c /light-api/input/serverConfig.json -o /light-api/out/hybridserver
```

In the Linux environment, the generated code might belong to root:root and you need to change the
owner to yourself before building it. 

Let's change the owner and build the server

```
cd /tmp/light-codegen
sudo chown -R steve:steve hybridserver
cd hybridserver
mvn clean install
```

Let's wait until we have a server built to start the server and test it.


#### Docker Scripting

```
git clone https://github.com/networknt/model-config.git
cd model-config
./generate.sh light-hybrid-4j-server ~/networknt/model-config/hybrid/generic-server /tmp/hybridserver
```

Now you should have a project generated in /tmp/hybridserver/generated

#### Codegen Site

The light-codegen service API is ready. We are working on the UI with a generation wizard.
 
[documentation]: /style/light-hybrid-4j/
[project]: https://github.com/networknt/light-hybrid-4j
[download or build]: /tool/light-codegen/download-build/
