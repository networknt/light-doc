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
---

In the previous step, we've updated project with H2 database to perform unit test
without any external database. In this step, we are going to run some performance
test with wrk in Docker to give you a ballpark how fast this database demo is. 

We are not going to run it on real testing server with service and database separated
on two physical machines. Everything here is on the same desktop with a 4 core/4 thread
i5 CPU.

Before we start the test, we need to bring up the server in test folder as we have
postgres configured. 

```
cd ~/networknt/light-example-4j/rest/swagger/database/test
mvn clean install exec:exec
```

To test the endpoint on your localhost, use the following command. You cannot
use localhost, so you have to find out your ip address. 

You can use ifconfig to find you local ip.

```
docker run --rm williamyeh/wrk -t4 -c50 -d30s --timeout 2s https://192.168.1.120:8443/v1/query
```

Here is the result.

```
Running 30s test @ https://192.168.1.120:8443/v1/query
  4 threads and 50 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.34ms    9.79ms 256.80ms   97.02%
    Req/Sec     3.66k     1.06k    8.05k    83.22%
  421266 requests in 30.08s, 58.05MB read
Requests/sec:  14004.53
Transfer/sec:      1.93MB
```

As we are using Postgres docker container and its maximum connection can only reach 100.
If you can increase the connection pool size, you can get even better performance.


So far you have gone through important area with database access, there are other topic
related to database but they are out of scope of this simple tutorial. If you have questions
on SQL db access or NoSQL db access, please take a look at other examples in [light-example-4j][]

[light-example-4j]: https://github.com/networknt/light-example-4j

