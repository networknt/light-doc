---
title: "Httpperf"
date: 2017-11-29T12:15:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Now let's see if these servers are performing with
[wrk](https://github.com/wg/wrk). To learn how to use it, please see my
article in tools [here][]

Assume you have wrk installed, run the following command.

```
wrk -t4 -c128 -d30s http://localhost:7001 -s pipeline.lua --latency -- /v1/data 1024

```
And here is what I got on my i5 desktop

```
wrk -t4 -c128 -d30s http://localhost:7001 -s pipeline.lua --latency -- /v1/data 1024
Running 30s test @ http://localhost:7001
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.00us    0.00us   0.00us    -nan%
    Req/Sec     5.52k     5.58k   20.40k    80.30%
  Latency Distribution
     50%    0.00us
     75%    0.00us
     90%    0.00us
     99%    0.00us
  349184 requests in 30.07s, 80.25MB read
  Socket errors: connect 0, read 0, write 0, timeout 256
Requests/sec:  11613.39
Transfer/sec:      2.67MB
```

Before starting the next step, please kill all four instances by Ctrl+C. And check in
the httpchain folder we just created and updated. 

[here]: /tool/wrk-perf/
