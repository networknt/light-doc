---
title: "Eventuate Troubleshooting"
date: 2018-03-31T05:06:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Event-driven frameworks are very complicated as there are many moving parts. The following describes some troubleshooting techniques that can help developers to find out where would be the problem if the application is not working. 

### Turn on Debug Log

Please refer [debug log][] for more details on changing debug level. 


### Kakfa Troubleshooting

When using Kafka in a docker container, it is a little inconvenient to use the command line tools within Kafka bin folder. First, we need to get into the Kafka container. 

Before running docker command to get into the container, we need to find out the container id for the particular service you are interested in with "docker ps" command. 

Replace the command line below {container id} with the id you just found. 

```
docker exec -it {container id} bash
```
Some of the containers are using Alpine Linux as the base image, and bash is not available. Use the sh command in this scenario. 

```
docker exec -it {container id} sh
```

Once you are inside the container, you can use Kafka tools in the bin directory.  For example, you can list all the topics to see if your application event topic is on Kafka.

```
cd bin
./kafka-topics.sh --zookeeper zookeeper:2181 --list
```



[debug log]: /development/best-practices/debug-log/
