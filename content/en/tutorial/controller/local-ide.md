---
title: "Local Controller in IDE"
date: 2021-10-14T14:20:43-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this step, we will start confluent platform, light-scheduler, light-controller and petstore in IDE to use local configuration and debug three applications together during the development.

### Confluent Platform

```
cd ~/networknt/kafka-sidecar
docker-compose up

```

### Light-Scheduler

Start the light-scheduler with the config/local config folder.

### Light-Controller

Start the light-controller with the config/local config foler.

### Petstore

Start petstore from light-example-4j/rest/petstore-maven-single in IDE with default configuration.

### Test

From the browser, enter the URL.

```
https://localhost:8438

```

### Video


{{< youtube VI3WfYJ_arg >}}





