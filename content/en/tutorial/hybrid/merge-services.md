---
title: "Merge Multiple Services"
date: 2019-04-08T22:14:09-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to start a server with multiple services. Each service will have its schema.json, and they need to be merge during server startup so that validation and scope verification can be done on all requests based on the merged schema.

### Prepare Environment

Before starting, we need to prepare the environment by clone several repositories from networknt and build the light-codegen. Let’s assume that you are using a workspace called networknt under your user's home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/model-config.git
git clonehttps://github.com/networknt/light-example-4j.git
cd light-codegen
mvn clean install
cd ..
```

As we are going to regenerate a server and several services in light-example-4j, let’s rename the merge-schema folder so that you can compare them if you want.

```
cd ~/networknt/light-example-4j/hybrid
mv merge-schema merge-schema.bak
```

### Generate Merge Server

In light-codegen, the light-hybrid-4j framework generator needs a config.json as input to generate a server project. This file can be found in model-config/hybrid/merge-schema/server.

Here is the content of config.json

```
{
  "rootPackage": "com.networknt.merger",
  "handlerPackage":"com.networknt.merger.handler",
  "modelPackage":"com.networknt.merger.model",
  "artifactId": "merger",
  "groupId": "com.networknt",
  "name": "merger",
  "version": "1.0.0",
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

Here is the command line to generate the server from the networknt folder.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-server -o light-example-4j/hybrid/merge-schema/server -c model-config/hybrid/merge-schema/server/config.json
```

Build the server

```
cd ~/networknt/light-example-4j/hybrid/merge-schema/server
mvn clean install
```

### Generate Two Services

Now let’s generate two services. For hybrid service generator, it needs a config.json and also a schema.json to define the interface/contract for the service.

These files can be found in model-config/hybrid/merge-schema/service1 and service2 folder.

Here is the list of command lines to generate hybrid services

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/hybrid/merge-schema/service1 -m model-config/hybrid/merge-schema/service1/schema.json -c model-config/hybrid/merge-schema/service1/config.json
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/hybrid/merge-schema/service2 -m model-config/hybrid/merge-schema/service2/schema.json -c model-config/hybrid/merge-schema/service2/config.json

```


### Update Two Services

Now, let's update the two services to return a JSON object in the handler. 

For service1, let's update the QuerySide.java to


```
package com.networknt.merger.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/service1/query/0.1.0")
public class QuerySide implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("{\"message\":\"Hello Query!\"}");
    }
}

```

For service2, let's update the CommandSide.java to 

```
package com.networknt.merger.handler;

import com.networknt.utility.NioUtils;
import com.networknt.rpc.Handler;
import com.networknt.rpc.router.ServiceHandler;
import java.nio.ByteBuffer;
import io.undertow.server.HttpServerExchange;

@ServiceHandler(id="lightapi.net/service2/command/0.1.0")
public class CommandSide implements Handler {
    @Override
    public ByteBuffer handle(HttpServerExchange exchange, Object input)  {
        return NioUtils.toByteBuffer("{\"message\":\"Hello Command!\"}");
    }
}

```

Let's build the service1 and service2

```
cd ~/networknt/light-example-4j/hybrid/merge-schema/service1
./mvnw clean install
cd ~/networknt/light-example-4j/hybrid/merge-schema/service2
./mvnw clean install
```

### Start the server with services

Now let’s start the server with services in the classpath.

```
cd ~/networknt/light-example-4j/hybrid/merge-schema/server
java -cp target/merger-1.0.0.jar:../service1/target/merger-1.0.0.jar:../service2/target/merger-1.0.0.jar com.networknt.server.Server
```

Now the server is up and running with two services.

### Test

Access the query service

```
curl -k -X POST https://localhost:8443/api/json -H 'content-type: application/json' -d '{"host":"lightapi.net","service":"service1","action":"query","version":"0.1.0","data":{"q1":"Hu","q2":"Steve"}}'
```

The response should be

```
{"message":"Hello Query!"}
```

Access the command service

```
curl -k -X POST https://localhost:8443/api/json -H 'content-type: application/json' -d '{"host":"lightapi.net","service":"service2","action":"command","version":"0.1.0","data":{"c1":"Hu","c2":"Steve"}}'

```

The response should be

```
{"message":"Hello Command!"}
```

### Summary

As you can see, both services have their own schema during development. After they are deployed to the server, these schemas are merged together to form a single service. All traffic to the server instance will be routed to the right handler by the light-hybrid-4j framework. 

This tutorial doesn't explain the details but only walk you through the steps. If you want to learn more, please visit [hybrid tutorial][]


[hybrid tutorial]: /tutorial/hybrid/

