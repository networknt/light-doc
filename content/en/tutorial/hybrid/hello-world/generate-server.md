---
title: "Generate Hello World Hybrid Server"
date: 2019-04-16T14:10:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

To generate a hybrid server, we only need a config.json to specify project related parameters as there is no specification for the server. The server itself is just a host for multiple services. 

The config.json can be accessed at https://github.com/networknt/model-config/tree/master/hybrid/hello-world/server

```json
{
  "rootPackage": "com.networknt.hello",
  "handlerPackage":"com.networknt.hello.handler",
  "modelPackage":"com.networknt.hello.model",
  "artifactId": "hello",
  "groupId": "com.networknt",
  "name": "hello",
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

As you can see, we want to generate a generic server project without extra dependencies. You can always add more dependencies later in the pom.xml file. 

The next step is to use the built light-codegen to scaffold a server project. For the usage of the light-codegen for the hybrid server, you can visit the [document][].

To generate the server project, run the following commands. 

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-server -o light-example-4j/hybrid/hello-world/server -c model-config/hybrid/hello-world/server/config.json
```

Now a new project is generated in the light-example-4j/hybrid/hello-world/server folder. You can build and start the server in the command line. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world/server
./mvnw install
```

After a successful build, you can start the server with the java command line from the server folder.

```
java -jar target/hello-1.0.0.jar
```

The server is now up and running; however, it is not functioning as there is no service deployed. From the console, you can see that the handlers size is zero. 

```
RpcStartupHookProvider: handlers size 0
```

Now, let's shut down the server by issuing CTRL+C on the terminal. We need to create build service1 that will be deployed to the server. First, the [server1 schema][] will need to be created, and then we can use it to scaffold service1 project. 

[document]: /tool/light-codegen/hybrid-server/
[server1 schema]: /tutorial/hybrid/hello-world/service1-schema/
