---
title: "Docker Access Local Kafka"
date: 2018-12-08T14:40:20-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

While working on one of the streams processing applications running in a Docker container to access local Kafka instance on the docker host, I had a lot of trouble to get the connection done right. 

First, I tried to use the host IP address 192.168.1.144:9092 to connect to the Kafka server and it doesn't work as expected. The issue is that it works in some instances but not others. 

Then, I switched to the host name freedom:9092 and the same issue is still there. It is not working properly as expected. 

After googling on the Internet, I found the solution. We basically cannot use the Docker network in this scenario. We have to use host network. 

After modifying the docker-compose file as below, everything works perfectly. 

```
# devnet local docker-compose
version: "2"
services:
  chainwriter:
    image: networknt/com.networknt.chainwriter-1.0.0:latest
    ports:
      - 8443:8443
    volumes:
      - ./local1/chain-writer:/config
    network_mode: host    
  chainreader:
    image: networknt/com.networknt.chainreader-1.0.0:latest
    ports:
      - 8444:8444
    volumes:
      - ./local1/chain-reader:/config
    network_mode: host    

# Networks
#
# networks:
#   localnet:
#     external: true

```

