---
title: "Config Server Process"
date: 2018-07-04T14:47:31-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true 
---

There are three major processes in the light-config-server design: 

### Key/Value/Secret Maintenance

Key/value/secret maintenance is very simple. The focus point is the fine-grained authorization on who can access what value or secret. Once the values or secrets are in the database, they are ready to be retrieved when the first request comes in to get the config package. 

### Config Template Management

Config template management is a little bit tricky. We have a lot of config files from the light-4j platform that we can maintain in a repository in networknt for templates of each release version. For most organizations, they might have their own extension or customized middleware handlers with config templates. Most services will have their own config file templates as well. This requires that the config server can merge or overwrite config files from different levels. 

### Config Package Generation

For each version of framework, an environmental profile and a serviceId, there would be a package generated the first request comes in. Once it is generated, it should cache it locally so that the next request can just pick it up from the cache. 



