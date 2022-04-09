---
title: "Light Gateway Tutorial"
date: 2022-04-08T15:15:58-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

We have recently combined both light-proxy and light-router to create a new light-gateway project. Like the http-sidecar, which is based on the same codebase and used as a sidecar in a Kubernetes cluster, the light-gateway can be used in Docker, Windows or Linux services. 

The following tutorials demonstrate the usage of the light gateway. We use two demo services to complete the full cycle: market service and petstore service.

Both services are located in the light-example-4j project.  You need to check out the repo and build both locally. 

Also, we need to check out the light-gateway project in your workspace. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-gateway.git
```

Build market service

```
cd ~/networknt/lgight-exmaple-4j/rest/market
mvn clean install -Prelease
```

Build petstore service

```
cd ~/networknt/light-example-4j/rest/petstore-maven-single
mvn clean install -Prelease
```

Build light-gateway

```
cd ~/networknt/light-gateway
mvn clean install
```

* Use light gateway as [server proxy][] on the same host. 


[server proxy]: /tutorial/gateway/server-proxy/

