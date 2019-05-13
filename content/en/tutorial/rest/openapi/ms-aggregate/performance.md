---
title: "Aggregate Performance"
date: 2018-03-11T16:00:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Now let's see if these servers are performing with [wrk][]. To learn how to use it, please see my
article in tool [here][].

Assume you have wrk installed, run the following command.

```
wrk -t4 -c128 -d30s https://localhost:7441 -s pipeline.lua --latency -- /v1/data 1024

```

And here is what I got on my i5 desktop

```
steve@joy:~$ wrk -t4 -c128 -d30s https://localhost:7441 -s pipeline.lua --latency -- /v1/data 1024
Running 30s test @ https://localhost:7441
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.00us    0.00us   0.00us    -nan%
    Req/Sec     3.37k     3.45k   19.33k    84.81%
  Latency Distribution
     50%    0.00us
     75%    0.00us
     90%    0.00us
     99%    0.00us
  234560 requests in 30.08s, 55.70MB read
  Socket errors: connect 0, read 0, write 0, timeout 128
Requests/sec:   7797.23
Transfer/sec:      1.85MB
```

Before starting the next step, please kill all four instances by Ctrl+C. And check in
the https folder we just created and updated. 

For more steps, please refer to [ms-chain][] tutorial or [service discovery][] tutorial.

[wrk]: https://github.com/wg/wrk
[here]: /tool/wrk-perf/
[ms-chain]: /tutorial/rest/swagger/ms-chain/
[service discovery]: /tutorial/common/discovery/
