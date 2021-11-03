---
title: "Api Endpoint"
date: 2021-10-11T21:53:39-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The specification for the light-scheduler is straightforward. There are two endpoints with schedulers as the path. 

The get endpoint will get all the scheduled task definitions. 
The post endpoint will create/update/delete the task definitions.


### Specification

You can find the spec at [model-config](https://github.com/networknt/model-config/tree/master/rest/light-scheduler) repository. 


### Get Endpoint


```
curl -k --location --request GET 'https://localhost:8401/schedulers'
```

### Post Endpoint


Create

```
curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
  "host": "lightapi.net",
  "name": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
  "action": "INSERT",
  "frequency": {
    "timeUnit": "SECONDS",
    "time": 10
  },
  "topic": "controller-health-check",
  "start": 1633813345308,
  "data": {
    "tlsSkipVerify": "true",
    "address": "172.17.0.1",
    "lastExecuteTimestamp": "0",
    "executeInterval": "0",
    "lastFailedTimestamp": "0",
    "protocol": "https",
    "port": "8443",
    "interval": "10000",
    "id": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
    "deregisterCriticalServiceAfter": "120000",
    "healthPath": "/health/",
    "tag": "dev",
    "serviceId": "com.networknt.petstore-3.0.1"
  }
}'

```


Delete using different frequency will return a not found error. 

```
curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
  "host": "lightapi.net",
  "name": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
  "action": "DELETE",
  "frequency": {
    "timeUnit": "MINUTES",
    "time": 1
  },
  "topic": "controller-health-check",
  "start": 1633813345308,
  "data": {
    "tlsSkipVerify": "true",
    "address": "172.17.0.1",
    "lastExecuteTimestamp": "0",
    "executeInterval": "0",
    "lastFailedTimestamp": "0",
    "protocol": "https",
    "port": "8443",
    "interval": "10000",
    "id": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
    "deregisterCriticalServiceAfter": "120000",
    "healthPath": "/health/",
    "tag": "dev",
    "serviceId": "com.networknt.petstore-3.0.1"
  }
}'
```

The result would be 

```
{"statusCode":404,"code":"ERR11637","message":"OBJECT_NOT_FOUND","description":"Object task definition not found for key lightapi.net com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443 MINUTES.","severity":"ERROR"}
```



Delete

```

curl -k --location --request POST 'https://localhost:8401/schedulers' \
--header 'Content-Type: application/json' \
--data-raw '{
  "host": "lightapi.net",
  "name": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
  "action": "DELETE",
  "frequency": {
    "timeUnit": "SECONDS",
    "time": 10
  },
  "topic": "controller-health-check",
  "start": 1633813345308,
  "data": {
    "tlsSkipVerify": "true",
    "address": "172.17.0.1",
    "lastExecuteTimestamp": "0",
    "executeInterval": "0",
    "lastFailedTimestamp": "0",
    "protocol": "https",
    "port": "8443",
    "interval": "10000",
    "id": "com.networknt.petstore-3.0.1|dev:https:172.17.0.1:8443",
    "deregisterCriticalServiceAfter": "120000",
    "healthPath": "/health/",
    "tag": "dev",
    "serviceId": "com.networknt.petstore-3.0.1"
  }
}'
```


### Video Tutorial

{{< youtube sm_CZ86HSpA >}}

