---
title: "Light Eventuate 4j"
date: 2017-11-05T11:46:59-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 40
weight: 40
sections_weight: 40
aliases: []
toc: false
draft: false
---

When building services with light platform, you can use request/response approach
like [rest][], [graphql][] or [hybrid][] to serve consumers. For service to service
communication, you have two options. One is synchronous request/response over HTTP
This requires all involved services to be responsive and highly available. 
There is another way to handler interaction between services - asynchronous event driven.
light-eventuate-4j is not only event driven framework, it supports Event Sourcing and
CQRS to ensure data consistency between services. 

If you are a developer to build services, then you don't need to clone the source code
of light-eventuate-4j. You can start all the services through a docker-compose. 

The docker-compose file is in [light-docker][] repo. 

Assume that you have a workspace named networknt under your home directory and you have
docker installed on your machine. 

Before using 
```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-eventuate-cdc.yml up
```

The above docker-compose will start Kafka, Zookeeper, Mysql and CDC server at the same time.




[light-docker]: https://github.com/networknt/light-docker

