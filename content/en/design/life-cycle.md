---
title: "Server Life Cycle"
date: 2021-10-05T10:55:27-04:00
description: ""
categories: []
keywords: []
slug: ""
menu:
  docs:
    parent: "design"
    weight: 30
weight: 30
toc: false
draft: false
---

This page describes the lifecycle of light-4j API. The light-4j API follows a defined lifecycle, starting in the API Server Start phase, moving through API Running phase, and then 
when the API server shutdown, moving  API to shutdown phase.

- API Server Start

- API Running

- API Server Shutdown

The defined lifecycle applies for all API running approaches. It includes start API by command line or IDE, run API in docker container, or run API in Kubernetes/Openshift container environment.

### API Server Start Phase

At the API start phase, it will follow the steps below to start API server:

- Load Server config (base on server.yml configuration file)

- Register Server module

- Start a Daemon thread for API graceful shutdonw

- Run the StartupHookProvider chain (defined in service.yml)

- Initial middleware handlers

- Bind Server
 
  This task includes binding server port, load SSL certs (keystore and truststore), API service registration, and server start.




### API Running Phase

After API server started, the API move to Running phase and ready to handler request and return response.

The request will go through the middleware handler chain then reach the business handler.

please refer to  [cross-cutting-concerns][] for the detail,




### API Server Shutdown Phase

In the pre-container world, most applications ran on VMs or physical machines. If an application crashed, it took quite a while to boot up a replacement. If you only had one or two machines to run the application, this kind of time-to-recovery was unacceptable.

Instead, it became common to use process-level monitoring to restart applications when they crashed. If the application crashed, the monitoring process could capture the exit code and instantly restart the application.

Light-4j API server define a Daemon thread to receive SIGTERM signal and handle the API Server graceful showdown after the  SIGTERM signal received.

In API Server Shutdown phase, it will follow the steps to shutdown API server:

- deregister the API service
  
- run Undertow server gracefulShutdownHandler

- run the ShutdownHookProvider chain (defined in service.yml)

- stop server

So if user wants the API server handle something in the API Server Shutdown phase, simply define the logic in the ShutdownHookProvider and add it to ShutdownHookProvider chain in service.yml (or values.yml)

For receiving SIGTERM signal, there are three scenarios  based on how API server started:

#### API server start from command line or IDE

ctrl-C will send SIGTERM signal to VM.

From IDE,  normally there are two buttons to start the API, "Stop" and "Exit".

"Stop" will stop the VM directly and will not send  SIGTERM signal.

"Exit" support graceful Shutdown and will send  SIGTERM signal before stop the VM

-------

####API server start in docker (docker run or docker-compose up)

Both ctrl-C and "docker stop" support  graceful Shutdown and will send  SIGTERM signal before terminating  the docker container

----

#### API server start in kubernetes container

When Kubernetes stop the pod which contain the API container, it will send SIGTERM signal.

The Kubernetes termination lifecycle:

- Pod is set to the “Terminating” State and removed from the endpoints list of all Services

  At this point, the pod stops getting new traffic. Containers running in the pod will not be affected.


-  preStop Hook is executed

   If your application doesn’t gracefully shut down when receiving a SIGTERM you can use this hook to trigger a graceful shutdown. Most programs gracefully shut down when receiving a SIGTERM, but if you are using third-party code or are managing a system you don’t have control over, the preStop hook is a great way to trigger a graceful shutdown without modifying the application

   For example, define an endpoint call shutdown in your application and add the lifecycle section into kubernates deployment yaml:

```json
lifecycle:
      preStop:
        httpGet:
          port: 8080
          path: shutdown
```
- SIGTERM signal is sent to the pod
  
  At this point, Kubernetes will send a SIGTERM signal to the containers in the pod. This signal lets the containers know that they are going to be shut down soon.
  Your code should listen for this event and start shutting down cleanly at this point. This may include stopping any long-lived connections (like a database connection or WebSocket stream), saving the current state, or anything like that.
  Even if you are using the preStop hook, it is important that you test what happens to your application if you send it a SIGTERM signal, so you are not surprised in production!

- Kubernetes waits for a grace period

  At this point, Kubernetes waits for a specified time called the termination grace period. By default, this is 30 seconds. It’s important to note that this happens in parallel to the preStop hook and the SIGTERM signal. Kubernetes does not wait for the preStop hook to finish.

- SIGKILL signal is sent to pod, and the pod is removed

  If the containers are still running after the grace period, they are sent the SIGKILL signal and forcibly removed. At this point, all Kubernetes objects are cleaned up as well.


#### One important Note here

If the Dockerfile start the API server by shell bash, for example:

```json

FROM openjdk:11.0.3-slim
COPY /target/server.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

The shell (sh) does not pass received signals to its child processes by default. The suggested alternative was to run the internal command as an exec which would basically replace the parent process (sh) with the child

Change the CMD to below will allow API process receive the  SIGTERM signal

```json
CMD ["/bin/sh","-c","exec java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```


----
There are some reference documents about the graceful shutdown:

https://dzone.com/articles/gracefully-shutting-down-java-in-containers

https://learnk8s.io/graceful-shutdown

[cross-cutting-concerns]: /concern/
