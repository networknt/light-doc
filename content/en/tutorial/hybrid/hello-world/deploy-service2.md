---
title: "Deploy Service2"
date: 2019-04-04T10:26:47-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [tested the service2][] to make sure that it is working for the tests in the development project. In this step, we are going to build the jar file and deploy it to the server instance along with service1. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world/service2
./mvnw install
```

Once the build is done, you should have service2-1.0.0.jar in the target folder. We deploy the service1 by adding it to the classpath when starting the server instance, it works if you have one or two services, but if you have too many services or plan to add more services in the future, it is easier just to copy all services to a folder and put that folder as part of server classpath. 

Let's create a folder in the `/tmp` and name it service after that copy the two jar files from service1 and service2 to this newly created folder.

```
cd /tmp
mkdir service
cd service
cp ~/networknt/light-example-4j/hybrid/hello-world/service1/target/service1-1.0.0.jar .
cp ~/networknt/light-example-4j/hybrid/hello-world/service2/target/service2-1.0.0.jar .
```

With the two files in the service folder, let's start the server with the following command to load services from the service folder. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world/server/target
java -cp hello-1.0.0.jar:/tmp/service/* com.networknt.server.Server
```

For the server console, you can see the following that indicates four handlers are loaded. 

```
RpcStartupHookProvider: handlers size 4
RpcStartupHookProvider add id lightapi.net/service2/command/0.1.1 map to com.networknt.hello.handler.CommandNew
RpcStartupHookProvider add id lightapi.net/service2/command/0.1.0 map to com.networknt.hello.handler.CommandOld
RpcStartupHookProvider add id lightapi.net/service1/info/0.1.0 map to com.networknt.hello.handler.Info
RpcStartupHookProvider add id lightapi.net/service1/query/0.1.0 map to com.networknt.hello.handler.Query
```

You can test the service1 and service2 with the following commands. 

```
curl -k -X POST https://localhost:8443/api/json -d '{"host":"lightapi.net","service":"service1","action":"query","version":"0.1.0","data":{"param1":"value1","param2":"value2"}}'

curl -k -X POST https://localhost:8443/api/json -d '{"host":"lightapi.net","service":"service2","action":"command","version":"0.1.0","data":{"source":"value1","target":"value2"}}'

curl -k -X POST https://localhost:8443/api/json -d '{"host":"lightapi.net","service":"service2","action":"command","version":"0.1.1","data":{"source":"value1","target":"value2"}}'

```

As you can see, two services are merged together and served by the same server instance. You can, later on, develop other services and put them into the same service folder and restart the server instance to load them.

There are also other options that will be explained in the [summary section][]. 

[tested the service2]: /tutorial/hybrid/hello-world/test-service2/
[summary section]: /tutorial/hybrid/hello-world/hybrid-summary/
