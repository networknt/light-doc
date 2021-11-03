---
title: "Local Controller Cluster with docker-compose"
date: 2021-10-14T14:20:49-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

For production, the light controller should be deployed in cluster mode with at least three instances. As it depending on the light-scheduler, we also need to deploy three instances of light-scheduler. To complete the function, we will use the [pet store service][] to register to the controller. 


### Start Confluent

```
cd ~/networknt/kafka-sidecar
docker-compose up

```

### Start Scheduler

```
cd ~/networknt/light-scheduler
docker-compose up

```

### Start Controller

```
cd ~/networknt/light-controller
docker-compose up

```

### Start Petstore

```
cd ~/networknt/light-example-4j/rest/petstore-maven-single
mvn clean install -Prelease
java -jar target/server.jar

```

### Test

Access the scheduler API to see there should be one task definition. Also you can see the health check task is created every 10 seconds.

```
curl -k https://localhost:8401/schedulers
curl -k https://localhost:8402/schedulers
curl -k https://localhost:8403/schedulers
```

Access the controller API to see there should be one registered service. 

```
curl -k https://localhost:8438/services
curl -k https://localhost:8439/services
curl -k https://localhost:8440/services

```

To access the lookup endpoint for a particular service on each node.

```
curl -k 'https://localhost:8438/services/lookup?serviceId=com.networknt.petstore-3.0.1&tag=dev'
curl -k 'https://localhost:8439/services/lookup?serviceId=com.networknt.petstore-3.0.1&tag=dev'
curl -k 'https://localhost:8440/services/lookup?serviceId=com.networknt.petstore-3.0.1&tag=dev'

```

To access all the health checks. 

```
curl -k https://localhost:8438/services/check
curl -k https://localhost:8439/services/check
curl -k https://localhost:8440/services/check

```

To get heath check for a particular check id.

```
curl -k 'https://localhost:8438/services/check/com.networknt.petstore-3.0.1%7Cdev:https:172.17.0.1:8443'
curl -k 'https://localhost:8439/services/check/com.networknt.petstore-3.0.1%7Cdev:https:172.17.0.1:8443'
curl -k 'https://localhost:8440/services/check/com.networknt.petstore-3.0.1%7Cdev:https:172.17.0.1:8443'

```

### Video


{{< youtube bAraQSo686g >}}


[pet store service]: https://github.com/networknt/light-example-4j/tree/release/rest/petstore-maven-single
