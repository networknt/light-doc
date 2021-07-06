---
title: "Securityperf"
date: 2017-11-29T08:08:59-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Since we have both https and OAuth2 involved, we are going to test the performance again. 

```
wrk -t4 -c128 -d30s -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgxMzAwNTAzNiwianRpIjoiN2daaHY2TS14UXpvVDhuNVMxODNodyIsImlhdCI6MTQ5NzY0NTAzNiwibmJmIjoxNDk3NjQ0OTE2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.FkFbPTRXZf045_7fBlEPQTn7rNoib54TYQeFzSjLmMkUjrfDsJZD6EnrsAquDpHt8GKQNqGbyPzgiNWAIYHgwPZvM-lHw_dv0KUKii3D0woaFBkqu4vYxqyImROBii0B38evxPAZVONWqUncL21592bFPHsxGCz5oHL2unLv-oIQklWxcILpMrSL_tf7nhXHSu1RkRhshxAiAHSSpBZnluu4-jqZdEFtc5U_YApToUrKkmI_An1op5-6rS_I-fMbSnSctUoDgg3RT4Zvw1HC-ZLJlXWRF5-FD4uQOAOgy_T7PI75pNiuh4wgOGgdIf48X-7-fDkEbla-cVLiuj3z4g" https://localhost:7441 -s pipeline.lua --latency -- /v1/data 1024
```

And result

```
Running 30s test @ https://localhost:7441
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.00us    0.00us   0.00us    -nan%
    Req/Sec     4.68k     4.11k   15.64k    56.64%
  Latency Distribution
     50%    0.00us
     75%    0.00us
     90%    0.00us
     99%    0.00us
  235520 requests in 30.10s, 54.13MB read
  Socket errors: connect 0, read 0, write 0, timeout 128
Requests/sec:   7824.81
Transfer/sec:      1.80MB
```

As you can see the performance dropped a little due to JWT verification. 

