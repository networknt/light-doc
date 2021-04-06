---
title: "Install Eventuate on Windows"
date: 2018-02-26T20:19:39-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "deployment"
    weight: 40
weight: 40
aliases: []
toc: false
draft: false
reviewed: true
---

There are some dependent services that need to be started in order to do the development of
light-eventuate-4j, light-tram-4j and light-saga-4j. On Linux and Mac, we usually start all
of them from a docker-compose as it is described in this [tutorial][]. 

For Windows users, AFAIK docker-compose doesn't work. The following is a manual process to
install Kafka, Zookeeper and CDC server for tram and CDC server for eventuate. In production,
at least Kafka and Zookeeper will be installed as a cluster on Linux bear metal or VMs.  

### Start Kafka and Zookeeper

The following link is the official installation document from Kafka. 

https://kafka.apache.org/quickstart

To start Zookeeper and Kafka from Windows local, please use Windows console and type following commands

Start zookeeper:

```
> cd \\kafka_2.11-0.10.2.0\bin\window
> zookeeper-server-start ../../config/zookeeper.properties

```

Start Kafka

```
> cd \\kafka_2.11-0.10.2.0\bin\window
> kafka-server-start ../../config/server.properties

```


### Mysql Database

Next you need to install Mysql Database on your local. 

https://dev.mysql.com/doc/mysql-getting-started/en/#mysql-getting-started-installing

Once the database is up and running, please execute the following db script from MySql command line or MySql Workbench:

https://github.com/networknt/light-docker/blob/master/mysql/eventuate.sql

If you want to try the examples, you need to execute all the scripts in that folder.

In order to use Mysql binlog for CDC server, you need to copy the replication.cnf file
to /etc/mysql/conf.d folder and restart the server.  

https://github.com/networknt/light-docker/blob/master/mysql/replication.cnf


### Start CDC server for tram

If you want to start a tram CDC server from java -jar, you need to check out the following repos. I am assuming you are using networknt as your workspace under your user home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-tram-4j.git
cd light-tram-4j/tram-cdc-mysql-server/
mvn clean install
java -java target/tram-cdc-mysql-server.jar
```

### Start CDC server for eventuate

```
cd ~/networknt
git clone https://github.com/networknt/light-eventuate-4j.git
cd light-eventuate-4j/eventuate-cdc-server/
mvn clean install
java -java target/eventuate-cdc-server.jar

```

If all the above services are up and running, you can build and run integration tests for light-tram-4j and light-eventuate-4j and build applications based on these frameworks. 

[tutorial]: /tutorial/eventuate/getting-started/
