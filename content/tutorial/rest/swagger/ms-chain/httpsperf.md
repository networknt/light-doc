---
title: "Httpsperf"
date: 2017-11-29T16:16:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
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
    Req/Sec     5.29k     5.53k   21.08k    78.43%
  Latency Distribution
     50%    0.00us
     75%    0.00us
     90%    0.00us
     99%    0.00us
  288256 requests in 30.09s, 66.25MB read
  Socket errors: connect 0, read 0, write 0, timeout 256
Requests/sec:   9578.74
Transfer/sec:      2.20MB
```

As you can see https connections are slower than http connections which is expected. But is 
is not that slow compare with http. 

Before starting the next step, please kill all four instances by Ctrl+C. And check in
the httpschain folder we just created and updated. 

[here]: /tool/wrk-perf/
