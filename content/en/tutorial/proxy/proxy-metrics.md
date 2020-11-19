---
title: "Proxy Metrics"
date: 2020-11-18T16:45:44-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The following tutorial will enable the metrics handler on light-proxy. 

### Backend API

Let's start the backend API first by following the [OpenAPI Backend][]. We only need to start the first instance with port 8081 for this tutorial as load balancing is not a concern for this tutorial. 


### InfluxDB

Before starting the proxy server, we need to have a local InfluxDB instance. Let's start the InfluxDB with Docker by following the [Petstore Metrics][] tutorial. 

### Light Proxy

Let's put the light-proxy configuration files into the light-config-test/light-proxy/proxy-metrics folder. You can check out the light-config-test to your workspace like `~/networknt`. 

##### Config

First, we copy the config files from the openapi-proxy in the same parent folder to the proxy-metrics folder. 

Since we are only using one backend instance, so we need to update the proxy.yml file to leave only one instance for the target server.

proxy.yml

```
# Target URIs
hosts: https://localhost:8081
```

##### Start

Follow the [OpenAPI Proxy][] tutorial to start the light-proxy with config files in the proxy-metrics folder. 

```
cd ~/networknt/light-proxy
mvn clean install -DskipTests
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-proxy/proxy-metrics -jar target/light-proxy.jar
```

##### Test

Test the proxy and backend with the following command.

```
curl -k https://localhost:8080/v1/getData
```
```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTkyMTEwMTY4MCwianRpIjoiX1A5Qk05Sm5TS3I1WWM2ajZHcUw3ZyIsImlhdCI6MTYwNTc0MTY4MCwibmJmIjoxNjA1NzQxNTYwLCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTczIiwic2NwIjpbInByb3h5LnIiLCJwcm94eS53Il19.Oi1NEF4AVvgyVoVg4mqCVabT-ywdgNE_vlgRob6xmME3C9TXs7r7yWumWpyfQ2fMUfAS9pe3xvVff9mcFN6PMmhZrxVSIlzme9r-lxAsFgMSBjzPfUTZtxHpMb26WpD70zLAqW0dUhLV_k_tBUfnLnzcmqoh446koLDA7z6_cAhruA-nBg794lY1PZbb5Nz1b-WqfFoORnkYuYnYyUpl0loTrLWj_E6svE1QNf-aorJzSeptAAQ0W7SumXTeycNDnfFBc_qQmILmnjG5ayZrhEyFQnVqCdqzqmjBiu6sewgcjmmItdiuIgM1cJXshhPWrSUWpDbjNxCVNysQ5lQREw" https://localhost:8080/v1/getData
```

Result

```
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8081,"key":"key1"}
```

##### Metrics

Let's add a values.yml to the config folder in the light-config-test/light-proxy/proxy-metrics folder.

values.yml

```
metrics.enabled: true
```

We need to enable metrics handler in the handler.yml file and wire it into the request/response chain. 

```
handlers:
  # Light-framework cross-cutting concerns implemented in the microservice
  - com.networknt.exception.ExceptionHandler@exception
  - com.networknt.metrics.MetricsHandler@metrics


chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    # - cors
    - header
    # - path
    - specification
    - security
    #- body
    #- audit
    #- sanitizer
    - validator
    - proxy

```

Restart the light-proxy server again and send the test request. 

### Chronograf

After sending several requests, to go the site http://localhost:8888 and check the tables. 

![proxy-metrics](/images/proxy-metrics.png)


[OpenAPI Backend]: /tutorial/proxy/openapi-backend/
[Petstore Metrics]: /tutorial/rest/openapi/petstore/metrics/
[OpenAPI Proxy]: /tutorial/proxy/openapi-proxy/