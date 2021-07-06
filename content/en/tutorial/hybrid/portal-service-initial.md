---
title: "Portal Hybrid Service initialization"
date: 2018-03-21T06:46:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-hybrid-4j is a generic RPC framework, and it supports JSON RPC at the moment. In the future, we are going to support binary RPC like gRPC or other binary protocols.

Light-portal build based the the light-hybrid-4j framework. All services in the light-portal provide hybrid services for as microservice API.

For the detail of the light-hybrid-4j framework, please refer to:

https://github.com/networknt/light-hybrid-4j


To the light-portal on you local environment, please follow the following steps:



### Build environment

light-portal hybrid service has two separate part:

-- hybrid-command-service

-- hybrid-query-service



Build light-portal hybrid-command and hybrid-query base services:

```
cd ~/networknt/light-portal
mvn clean install
```

Copy the service based jar files to the hybrid-command and hybrid-query in light-config-test service folder:



```
cd ~/networknt/light-config-test/light-portal/hybrid-command/service
cp ~/networknt/light-portal/hybrid-command/target/hybrid-command-1.5.11.jar .

cd ~/networknt/light-config-test/light-portal/hybrid-query/service
cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.5.11.jar .
cp ~/networknt/light-portal/common-util/target/common-util-0.1.0.jar .
```


Then we have the base hybrid-command and hybrid-query service ready. Following other steps for light-portal build-in hybrid services:

 -- host-menu

 -- schema-form

 -- user-management


And the user can create their own hybrid services and copy it to light-config-test hybrid  service folders.


