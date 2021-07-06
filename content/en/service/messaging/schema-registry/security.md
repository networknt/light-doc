---
title: "Schema Registry Security"
date: 2018-11-30T16:54:55-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When you want to expose Schema Registry access to client applications in your network or even the Internet, you need to make sure that Authentication and Authorization are properly configured so that only authorized users/apps can update schema and most users/apps will just have read-only access. 

Confluent has a paid security plugin as part of the enterprise license for authorization, but most of our users might not have access to it. Here we are providing a simple solution with [light-proxy][]. 

We can deploy/install the schema registry on one of the VMs and lock it down to allow only light-proxy VM to access to it on port 8081. On light-proxy instance, we can add those endpoints that are read-only from schema registry into the handler.yml config file. It allows the public Internet to read the Avro schema, but the internal application can write into it. 

Here is a [tutorial][] to show you how to config the light-proxy instance to do that. 

[light-proxy]: /service/proxy/
[tutorial]: /tutorial/proxy/schema-registry/
