---
title: "Light Hybrid 4j"
date: 2017-11-05T11:46:42-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 30
weight: 30
sections_weight: 30
aliases: []
toc: false
draft: false
reviewed: true
---

For public APIs, it makes sense to use RESTful; however, if it is an internal API, the RPC based API style will be more efficient. As these days, Javascript on the browser is really powerful, and it can deal with JSON or other binary protocol very easily. These browser object (JSON/Binary) contains type information so that there is no need to do Object to URI and URI to Object transformation based on OpenAPI specification or RAML.

If you look at an OpenAPI specification, you will be shocked as it is too complicated with so many moving parts. Both client and server will be dealing with headers, query parameters, path parameters, and request body. Also, because of the transformation between JSON object to URI, data type information is lost, and you have to define it in the specification so that the server can derive the type from it. It makes the REST framework less efficient than other RPC based frameworks that send the JSON or binary object to the server with all the type information available. 

With hybrid schema, it is very easy to combine several services together as merging schemas are much easier than OpenAPI specification. This is why we call the light-hybrid-4j framework as a hybrid. You can put several services together into the same JVM instance to save memory and later on you can split up high volume service to separate JVM instance for scaling. This gives customers more flexible deployment options, and they can move from monolithic->hybrid-> microservices based on the growth of the business. 

Light-hybrid-4j is a generic RPC framework, and it supports JSON RPC.  Other binary RPC like gRPC will be supported later. 

The easiest way to start a hybrid service is too use [light-codegen][] to generate a server and then generate a service based on a schema. Then you can put the service in the pom.xml of the server, or put the service jar file into the classpath when starting the server. The second options is more flexible than the first one. 

To show users how easy to start a generic server and build a generic service, let's follow the steps below.

### Prepare Environment

Before starting, we need to prepare the environment by clone several repositories from the networknt organization on GitHub.com and build light-codegen. Let’s assume that you are using a workspace called networknt under your user's home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/model-config.git
git clonehttps://github.com/networknt/light-example-4j.git
cd light-codegen
mvn clean install
cd ..
```

As we are going to regenerate the generic-server and generic-service in light-example-4j, let’s rename these folders so that you can compare them if you want.

```
cd ~/networknt/light-example-4j/hybrid
mv generic-server generic-server.bak
mv generic-service generic-service.bak
```

### Generate Generic Server

It leverages the generic server and generic service defined in model-config/hybrid. These server and service are used to test the new version of the light-hybrid-4j and new version of light-codegen.

In light-codegen, light-hybrid-4j framework generator needs a config.json as input to generate a server project. This file can be found in model-config/hybrid/generic-server

Here is the content of config.json

```
{
  "rootPackage": "com.networknt.gserver",
  "handlerPackage":"com.networknt.gserver.handler",
  "modelPackage":"com.networknt.gserver.model",
  "artifactId": "gserver",
  "groupId": "com.networknt",
  "name": "gserver",
  "version": "1.0.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": false,
  "supportPostgresql": false,
  "supportH2ForTest": false,
  "supportClient": false
}

```

Here is the command line to generate server from the networknt workspace.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-server -o light-example-4j/hybrid/generic-server -c model-config/hybrid/generic-server/config.json
```

Build the generic server

```
cd ~/networknt/light-example-4j/hybrid/generic-server
mvn clean install
```

### Generate Generic Service

Now let’s generate a service. For hybrid service generator, it needs a config.json and also a schema.json to define the interface/contract for the service.

Service config.json can be found in model-config/hybrid/generic-service, and its content is

```
{
  "rootPackage": "com.networknt.gservice",
  "handlerPackage":"com.networknt.gservice.handler",
  "modelPackage":"com.networknt.gservice.model",
  "artifactId": "gservice",
  "groupId": "com.networknt",
  "name": "gservice",
  "version": "1.0.0",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportOracle": false,
  "supportMysql": false,
  "supportPostgresql": false,
  "supportH2ForTest": false,
  "supportClient": false
}

```

The schema.json can be found in the same folder. 

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

Here is the command line to generate the generic service

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/hybrid/generic-service -m model-config/hybrid/generic-service/schema.json -c model-config/hybrid/generic-service/config.json
```

Once the service is generated, let's update the HelloWorld2.java to return a JSON object instead of an empty string. 

```
package com.networknt.gservice.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/world/hello/0.1.1")
public class HelloWorld2 implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("{\"message\":\"Hello World!\"}");
    }
}

```


Build generic service

```
cd ~/networknt/light-example-4j/hybrid/generic-service
mvn clean install
```

### Start the Server with Service

Now let’s start the server with service in the classpath.

```
cd ~/networknt/light-example-4j/hybrid
java -cp generic-server/target/gserver-1.0.0.jar:generic-service/target/gservice-1.0.0.jar com.networknt.server.Server
```

Now the server is up and running with 4 handlers.

### Test

Issue the following command.

```
curl -k -X POST https://localhost:8443/api/json -H 'content-type: application/json' -d '{"host":"lightapi.net","service":"world","action":"hello","version":"0.1.1","data":{"lastName":"Hu","firstName":"Steve"}}'
```

You will have a response like this.

```
{"message":"Hello World!"}
```

### Summary

This getting started guide shows you the steps without any extra options. You can get a feeling of how the light-hybrid-4j framework works. If you are interested in this framework, please visit the [tutorial][]. 

[light-codegen]: /tool/light-codegen/
[tutorial]: /tutorial/hybrid/
