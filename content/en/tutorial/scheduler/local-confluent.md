---
title: "Local Confluent"
date: 2021-10-12T07:19:35-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

There are two ways to start local confluent platform to support local development and testing. 


### Local Installation

If you work on a Linux desktop, download and install the confluent platform is the easiest way, and all the command line tools will be available with the command line on your desktop. You can use a shell script to create/delete resources.


```
confluent local services start
```


### Docker Compose

If you have docker installed on your desktop, you can start the confluent platform locally with docker-compose. The docker-compose.yml is available in the kafka-sidecar repository.


```
cd ~/networknt/kafka-sidecar
docker-compose up
```

### Video Tutorial

{{< youtube L5RoJvCbkT0 >}}

