---
title: "Hybridservice"
date: 2017-11-22T15:24:44-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


This is a generator that scaffold hybrid service which is on top of light-hybrid-4j and light-4j
frameworks. Hybrid framework contains a set of handlers to support RPC style of contract instead
of Restful to eliminate the complex transformation between model and URI and vice verse. Currently
we support JSON RPC only and binary RPC will be supported if there are requests. Another benefit
of hybrid framework is that services can be composed as a modular monolith and running within the
same JVM in the beginning and then split them into several JVM later on if load is getting higher.

It is very suitable for small and medium size business as you can start microservices without
the huge front end infrastructure investment. 

## Input

#### Model

As light-hybrid-4j is an RPC based framework, only HTTP POST method is used. Currently, only JSON based RPC is
implemented and binary formatted protocol will follow. 

Here is an exmaple of schema that defines several services. 

```
{
  "host": "lightapi.net",
  "service": "world",
  "action": [
    {
      "name": "hello",
      "version": "0.1.0",
      "handler": "HelloWorld1",
      "scope" : "world.r",
      "schema": {
        "title": "Service",
        "type": "object",
        "properties": {
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          },
          "age": {
            "description": "Age in years",
            "type": "integer",
            "minimum": 0
          }
        },
        "required": ["firstName", "lastName"]
      }
    },
    {
      "name": "hello",
      "version": "0.1.1",
      "handler": "HelloWorld2",
      "scope" : "world.r",
      "schema": {
        "title": "Service",
        "type": "object",
        "properties": {
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          },
          "age": {
            "description": "Age in years",
            "type": "integer",
            "minimum": 0
          }
        },
        "required": ["firstName", "lastName"]
      }
    },
    {
      "name": "hi",
      "version": "0.0.1",
      "handler": "HiWorld",
      "scope" : "world.r",
      "schema": {
        "title": "Service",
        "type": "object",
        "properties": {
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          },
          "age": {
            "description": "Age in years",
            "type": "integer",
            "minimum": 0
          }
        },
        "required": ["firstName", "lastName"]
      }
    },
    {
      "name": "welcome",
      "version": "0.0.1",
      "handler": "WelcomeWorld",
      "scope" : "world.w",
      "schema": {
        "title": "Service",
        "type": "object",
        "properties": {
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          },
          "age": {
            "description": "Age in years",
            "type": "integer",
            "minimum": 0
          }
        },
        "required": ["firstName", "lastName"]
      }
    }
  ]
}

```


#### Config

Here is an exmaple of config.json for light-hybrid-4j generator.

```
{
{
  "name": "petstore",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "petstore",
  "rootPackage": "com.networknt.petstore",
  "handlerPackage":"com.networknt.petstore.handler",
  "modelPackage":"com.networknt.petstore.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
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
}
```


- name is used in generated pom.xml for project name
- version is used in generated pom.xml for project vesion
- groupId is used in generated pom.xml for project groupId
- artifactId is used in generated pom.xml for project artifactId
- rootPackage is the root package name for your project and it will normally be your domain plug project name.
- handlerPackage is the Java package for all generated handlers. 
- modelPackage is the Java package for all generated models or POJOs.
- overwriteHandler controls if you want to overwrite handler when regenerate the same project into the same folder. If you only want to upgrade the framework to another minor version and don't want to overwrite handlers, then set this property to false. 
- overwriteHandlerTest controls if you want to overwrite handler test cases.
- httpPort is the http port number the server is listening to if it is enabled.
- enableHttp is the flag to control if http is enabled or not.
- httpsPort is the https port number the server is listening to if it is enabled.
- enableHttps is the flag to control if https is enabled or not.
- supportDb to control if db connection pool will be setup in service.yml and db dependencies are included in pom.xml
- dbInfo section is the database connection pool configuration info.
- supportH2ForTest is a flag to control if H2 code is included in the test server and H2 jar is included in pom.mxl
- supportClient is a flag to control if client module is included in the generated project to call other services.



## Usage

#### Java Command line

Before using the command line to generate the code, you need to check out the repo and build it.
I am using ~/networknt as workspace but it can be anywhere in your home.  

```
cd ~/networknt
git clone git@github.com:networknt/light-codegen.git
cd light-codegen
mvn clean install
```

This is a generator that scaffolds a service module that will be hosted on a light-hybrid-4j server
platform as a jar file. Multiple modules can be hosted on the same server if needed and they can
interact with each other through a module interface/contract defined by schema files. The generated
project cannot run directly as it's only a small jar file without a main class. However its services are
enabled by placing the jar file into the classpath of the server. For multiple jar files, we recommend
creating a folder like /service or /lib within the class path of the server and having it contain all
necessary light-hybrid-4j services.

The following will generate a sample light-hybrid-4j service which exposes a hello-world endpoint.
Working directory: light-codegen
```
java -jar codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o /tmp/hybridservice -m light-hybrid-4j/src/test/resources/schema.json -c light-hybrid-4j/src/test/resources/serviceConfig.json
```

Now we have a server and a service generated. Let's go through deploying the service and starting the server.

First let's build the **service** and copy the jar file into the **server** folder:
```
cd /tmp/hybridservice
mvn clean install
cp target/petstore-1.0.1.jar /tmp/hybridserver
```

Now let's build the **server** and start it while including the above **service** in its classpath:
```
cd /tmp/hybridserver
mvn clean install
java -cp petstore-1.0.1.jar:target/petstore-1.0.1.jar com.networknt.server.Server
```

Let's use curl to test one of the services:
```
curl -X POST \
  http://localhost:8080/api/json \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'postman-token: 58bb63eb-de70-b855-a633-5b043bb52c95' \
  -d '{
  "host": "lightapi.net",
  "service": "world",
  "action": "hello",
  "version": "0.1.1",
  "lastName": "Hu",
  "firstName": "Steve"
}'
```

#### Docker Command Line

The following command is using docker to generate light-hybrid-4j service into
/tmp/light-codegen/hybridservice folder

```
docker run -it -v ~/networknt/light-codegen/light-hybrid-4j/src/test/resources:/light-api/input -v /tmp/light-codegen:/light-api/out networknt/light-codegen -f light-hybrid-4j-service -m /light-api/input/schema.json -c /light-api/input/serviceConfig.json -o /light-api/out/hybridservice
```

On Linux environment, the generated code might belong to root:root and you need to change the
owner to yourself before building it. 

Let's change the owner and build the service

```
cd /tmp/light-codegen
sudo chown -R steve:steve hybridservice
cd hybridservice
mvn clean install

```

To run the server with services, please following the instruction in utility command line.

#### Docker Scripting

```
git clone git@github.com:networknt/model-config.git
cd model-config
./generate.sh light-hybrid-4j-service ~/networknt/model-config/hybrid/generic-service /tmp/hybridservice
```

Now you should have a project generated in /tmp/hybridservice/generated

#### Codegen Site

The service API is ready. We are working on the UI with a generation wizard.
 
