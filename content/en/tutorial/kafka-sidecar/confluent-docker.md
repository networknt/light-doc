---
title: "Confluent Docker"
date: 2021-08-31T17:48:19-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

If you have Docker installed on your desktop, you can use the local Kafka docker image to start the confluent platform. 

https://docs.confluent.io/platform/current/quickstart/ce-docker-quickstart.html


A docker-compose.yml is created in the root directory to start the confluent platform. 

```
cd ~/networknt/kafka-sidecar
docker-compose up -d
```

If you are using Linux desktop, you can start [confluent local](/tutorial/kafka-sidecar/confluent-local/) from the command line. 
