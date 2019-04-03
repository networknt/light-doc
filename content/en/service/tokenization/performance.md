---
title: "Tokenization Performance"
date: 2018-03-24T12:32:04-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The tokenization service is designed to be a hosting service that can support thousands of clients to access concurrently. If this service is installed in-house, it is supposed to be used in high throughput and low latency enterprise environment. The performance of the service is critical.

The following section described how the performance test is done locally to measure if the service is suitable for your use case. 

### Prepare Environment

A remote instance and tokenization instance are in a docker-compose file to start them together. The test case is designed to use the local cache as we only use one token to retrieve the value for the token. It is to mimic the real world scenario as the production environment should have a big cache for a small set of key-value pair. 

To start the service with docker locally. 

```
cd ~
mkdir networknt
cd networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-tokenization.yml up -d
```

### Performance Test

The performance test will only be focus on the detokenizer as it is the most concerns in real world scenario. The tokenizer is basically database throughput test. 

First let's create a token. 

```
curl -k -X POST https://localhost:8443/v1/token -H 'content-type: application/json' -H 'authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg' -d '{"schemeId":1,"value":"1234567890"}'
```

Write down the response and issue a command to get the value to verify it works. 

```
curl -k -H 'authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg' https://localhost:8443/v1/token/IR-8BAhST1ahni1UTdT8zw

```

Now use wrk.

```
wrk -t4 -c128 -d30s -H "authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg" https://localhost:8443 -s pipeline.lua --latency -- /v1/token/IR-8BAhST1ahni1UTdT8zw 1024
```

And here is the result on my aging i5 desktop with production configuration.  

```
steve@joy:~$ wrk -t4 -c128 -d30s -H "authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg" https://localhost:8443 -s pipeline.lua --latency -- /v1/token/IR-8BAhST1ahni1UTdT8zw 16
Running 30s test @ https://localhost:8443
  4 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    31.39ms   29.85ms 390.83ms   88.74%
    Req/Sec    13.32k     2.26k   20.36k    72.69%
  Latency Distribution
     50%   23.11ms
     75%   39.80ms
     90%   64.96ms
     99%  151.94ms
  1582528 requests in 30.06s, 185.63MB read
Requests/sec:  52649.86
Transfer/sec:      6.18MB
```

