---
title: "Generate Service1"
date: 2019-04-04T10:25:45-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have created [service1 schema][] file in the model-config repository. In this step, we are going to create a config.json file and generate the service1 with light-codegen. 

The config.json should be very similar to the config.json for the server. You can find it at https://github.com/networknt/model-config/tree/master/hybrid/hello-world/service1

config.json

```
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

For information on how to use light-codegen to scaffold hybrid service project, please visit [hybrid service][]

With both schema.json and config.json ready for the service1, let's run the following commands to scaffold the project. 

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-hybrid-4j-service -o light-example-4j/hybrid/hello-world/service1 -m model-config/hybrid/hello-world/service1/schema.json -c model-config/hybrid/hello-world/service1/config.json
```

After the generation command line is completed, a new project service1 should be generated in light-example-4j/hybrid/hello-world folder. 

Let's go to the service1 folder and build it. 

```
cd ~/networknt
cd light-example-4j/hybrid/hello-world/service1
./mvnw install
```

In the next step, we are going to update the service1 and [test it][] from the IntelliJ IDEA with unit/integration test cases.

[service1 schema]: /tutorial/hybrid/hello-world/service1-schema/
[hybrid service]: /tool/light-codegen/hybrid-service/
[test it]: /tutorial/hybrid/hello-world/test-service1/
