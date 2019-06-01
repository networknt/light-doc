---
title: "Performance"
date: 2017-11-24T17:32:53-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 110     #rem
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we've updated project with H2 database to perform the unit test without any external database. In this step, we are going to run some performance test with wrk in Docker to give you a ballpark how fast this database demo is. 

We are not going to run it on a real testing server with service and database separated on two physical machines. Everything here is on my desktop with an AMD 2700x CPU.

Before we start the test, we need to bring up the server in the test folder as we have the Postgres configured. 

```
cd ~/networknt/light-example-4j/rest/swagger/database/test
mvn clean install exec:exec
```

To test the endpoint on your localhost, use the following command. You cannot use localhost, so you have to find out your IP address. 

You can use `ifconfig` to find your local IP address. My local IP is 192.168.1.144 and you need to replace it with your IP in the following docker command line.

```
docker run --rm williamyeh/wrk -t4 -c50 -d30s --timeout 2s https://192.168.1.144:8443/v1/query
```

Here is the result.

```
steve@freedom:~$ docker run --rm williamyeh/wrk -t4 -c50 -d30s --timeout 2s https://192.168.1.144:8443/v1/query
Unable to find image 'williamyeh/wrk:latest' locally
latest: Pulling from williamyeh/wrk
4fe2ade4980c: Already exists 
c4d7e348633d: Pull complete 
3e403d3ebdda: Pull complete 
bdb672ee55d9: Pull complete 
2bfb714176a4: Pull complete 
Digest: sha256:78adc0d9d51a99e6759e702a08d03eaece81c890ffcc9790ef9e5b199d54f091
Status: Downloaded newer image for williamyeh/wrk:latest
Running 30s test @ https://192.168.1.144:8443/v1/query
  4 threads and 50 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.77ms    9.53ms 229.90ms   99.05%
    Req/Sec    13.37k     2.99k   16.01k    91.02%
  1569848 requests in 30.01s, 215.89MB read
Requests/sec:  52305.78
Transfer/sec:      7.19MB
```

As we are using Postgres docker container and its maximum connection can only reach 100. If you can increase the connection pool size, you can get even better performance.

So far you have gone through the important area with database access, there are other topics related to the database, but they are out of the scope of this simple tutorial. If you have questions on SQL DB access or NoSQL DB access, please take a look at other examples in [light-example-4j][]

[light-example-4j]: https://github.com/networknt/light-example-4j

