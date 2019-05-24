---
title: "Deploy Service1"
date: 2019-04-04T10:26:03-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [tested the service1][] within the IDE. In the current step, we are going to build the service1 and deploy it to the server instance. 

First, let's build the service1 from the command line. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world/service1
./mvnw install
```

After the build is done, you should find a jar file named service1-1.0.0.jar in the target folder. 

```
-rw-r--r--  1 steve steve  6223 Apr  4 23:58 service1-1.0.0.jar
```


You can see that the jar file is only 6K in size as there are only two handler classes in the jar. 

To deploy the service1 with the server instance, we just need to put the service1-1.0.0.jar in the classpath when starting the server instance. 

```
cd ~/networknt/light-example-4j/hybrid/hello-world/server
java -cp ../service1/target/service1-1.0.0.jar:target/hello-1.0.0.jar com.networknt.server.Server
```

Now the server should be started with two handlers loaded. You can find out from the server console with the following output. 

```
RpcStartupHookProvider: handlers size 2
RpcStartupHookProvider add id lightapi.net/service1/info/0.1.0 map to com.networknt.hello.handler.Info
RpcStartupHookProvider add id lightapi.net/service1/query/0.1.0 map to com.networknt.hello.handler.Query
```

To test the service1, send a request to the server instance with the following curl command. 

```
curl -k -X POST https://localhost:8443/api/json -d '{"host":"lightapi.net","service":"service1","action":"query","version":"0.1.0","data":{"param1":"value1","param2":"value2"}}'
```

The return value should be "query". This means that the service1 is successfully loaded by the server instance. This approach only works when you have one or two services. If you have too many services, you would better to create a folder and put all your services into it. When starting the server instance, you put the folder as part of the classpath. 

In the next step, let's create a new [service2 schema][] and we are going to build service2 and put both services into a folder to load by the server instance. 


[tested the service1]: /tutorial/hybrid/hello-world/test-service1/
[service2 schema]: /tutorial/hybrid/hello-world/service2-schema/

