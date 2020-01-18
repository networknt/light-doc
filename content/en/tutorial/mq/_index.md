---
title: "Light MQ Tutorial"
date: 2020-01-18T11:28:50-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For documentation on the light-mq component, please refer to [light-mq](/style/light-mq/)

The source code of the light-mq is not open-sourced; however, we have published two docker images and a docker-compose file that can be executed locally for any user. The docker-compose contains two services built on top of the light-rest-4j and light-mq. One service is a producer that can put the request body into an MQ queue and publish the request body to an MQ topic.  Another service is a consumer that has two startup hooks to consume the message from the MQ queue and the MQ topic and keep the last the messages from the queue and topic for users to access through the two endpoints. 

### Specification

The specification and light-codegen config.json for the producer is located at https://github.com/networknt/model-config/tree/master/rest/openapi/mq-producer

There are two post endpoints in the specification. 

The `/put` will put the key/value pair request body into the default queue `DEV.QUEUE.1` on the QueueManager `QM1` through channel `DEV.APP.SVRCONN`.

The `/pub` will publish the key/value pair request body to the default topic `dev/` o the QueueManager `QM1` through channel `DEV.APP.SVRCONN`.


The specification and light-codegen config.json for the consumer is located at https://github.com/networknt/model-config/tree/master/rest/openapi/mq-consumer

There are two get endpoints in the specification.

The `/get` will get up to ten key/value pairs the producer `/put` endpoint pushes to the MQ queue. 

The `/sub` will get up to ten key/value pairs the producer `/pub` endpoint pushes to the MQ topic. 

The corresponding README.md contains the command lines to scaffold the projects. 

### IBM MQ Docker

To start the demo applications on your local computer, you can install the IBM MQ on your local or simply use the docker image published by IBM for developers. 

Use the following command to start the docker container. We also set the password for the default user `app` so that we can connect to the MQ with authentication and authorization enabled. The light-mq component we provided doesn't allow connection to the MQ without a password 

```
docker run --env LICENSE=accept \
           --env MQ_QMGR_NAME=QM1 \
           --env MQ_APP_PASSWORD=passw0rd \
           --publish 1414:1414 \
           --publish 9443:9443 \
           --detach \
           ibmcom/mq
```

Once the docker container is up and runing, you can access the MQ console from the web browser with the following address. 

https://localhost:9443

As the docker container uses a self-signed certificate, you need to accept the risk and let the browser to access the local site. 

### Docker Compose

The docker-compose.yml for the producer and consumer examples can be found at https://github.com/networknt/light-config-test/tree/master/light-mq

Before starting the docker-compose, please make sure that the MQ docker is started. The example applications will connect the MQ server during the startup. 

To connect to the MQ server running in a docker container, we need to make sure that the host IP address is available for the producer and consumer to connect from the docker-compose containers. 

First, we need to find out your host IP address. On Linux, I am using `ifconfig`. 

On my desktop computer, the IP is 192.168.1.144, and you should have a similar address or something like 10.xxx.xxx.xxx if you are on a corporate network. 

Once you find the IP address, update the mq.yml in the producer and consumer folder with your IP address to replace the 192.168.1.144 for the host. 

The producer file can be located at https://github.com/networknt/light-config-test/blob/master/light-mq/producer/mq.yml

The consumer file can be located at https://github.com/networknt/light-config-test/blob/master/light-mq/consumer/mq.yml

After replacing the two mq.yml files with your host IP address, you can start the docker-compose as following. To make sure it works, you can use the `docker-compose up` without the `-d` to see the console output. You can also use `docker ps` to check if three docker containers are running to confirm all services are ok. 

```
cd ~/networknt
git clone https://github.com/networknt/light-config-test.git
cd light-config-test/light-mq
docker-compose up -d
```

### Test

At this moment, both the consumer and the producer are running. Let's send some requests to the producer `/put` endpoint. 

```
curl -k -X POST \
  https://localhost:8443/put \
  -H 'content-type: application/json' \
  -d '{"key":"key1", "value": "value1"}'
curl -k -X POST \
  https://localhost:8443/put \
  -H 'content-type: application/json' \
  -d '{"key":"key2", "value": "value2"}'
```

After waiting for 3 seconds, you can access the consumer service to get the data sent from the producer. 

```
curl -k https://localhost:8444/get
```

The result should be.

```
[{"key":"key1","value":"value1"},{"key":"key2","value":"value2"}]
```

Now, let's test the publish and subscription of the topic by issue a request to the `/pub` endpoint on the producer service. 


```
curl -k -X POST \
  https://localhost:8443/pub \
  -H 'content-type: application/json' \
  -d '{"key":"key3", "value": "value3"}'
curl -k -X POST \
  https://localhost:8443/pub \
  -H 'content-type: application/json' \
  -d '{"key":"key4", "value": "value4"}'

```

After waiting for 3 seconds, you can access the consumer service to get the data sent from the producer to the topic. 

```
curl -k https://localhost:8444/sub
```

The result should be.

```
[{"key":"key3","value":"value3"},{"key":"key4","value":"value4"}]
```

You can send more messages to the queue or the topic through the producer service; however, only the last ten messages will be kept in the consumer service to avoid extensive memory usage. In a real application, the consumer might process the data and persist the result into a database or invoke another service based on the processed result. It is always up to your business and developers' imagination.

