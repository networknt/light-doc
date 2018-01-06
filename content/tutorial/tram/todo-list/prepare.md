---
title: "Prepare Environment"
date: 2018-01-06T12:59:09-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In order to work with light-tram-4j, we need to start the infrastructure services and the easier
way is to use docker-compose. Here is a list of services that need to be started before we start
integration testing. 

* Kafka
* Zookeeper
* Mysql
* CDC service for light-tram-4j


To start Kafka, Zookeeper and Mysql, you can use the [docker-compose-eventuate.yml][] in light-docker
repository. And CDC server for light-tram-4j can be found at [docker-compose-cdcserver-for-tram.yml][] 

The detail steps are documented in the [getting started] in light-tram-4j

[docker-compose-eventuate.yml]: https://github.com/networknt/light-docker/blob/master/docker-compose-eventuate.yml
[docker-compose-cdcserver-for-tram.yml]: https://github.com/networknt/light-docker/blob/master/docker-compose-cdcserver-for-tram.yml
[getting started]: /getting-started/light-tram-4j/

