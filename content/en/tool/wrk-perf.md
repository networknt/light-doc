---
title: Wrk
date: 2017-11-04T22:12:19-04:00
description: "Using wrk for performance test"
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

For most API frameworks in Java, the server can only handle up to ten thousand requests per second, and Apache JMeter or Apache Bench will do the job. However, when we are talking about millions of requests per second light-4j handles, they won't work as these tools will use up all the CPU resources. For high-performance testing, [wrk](https://github.com/wg/wrk) is the best as it can send tens of millions of requests per seconds from commodity hardware.

Another limitation on high-performance throughput test is network limitation. Unless you client and server are connected with the 10G network, chances are the network will be the bottle net and you cannot reach the full handling potential of your HTTP server. To work around it, most of my tests are running on the same computer in order to gauge the raw server throughput and latency.

## Installation

[Install on Linux](https://github.com/wg/wrk/wiki/Installing-Wrk-on-Linux)

[Install on Mac](https://github.com/wg/wrk/wiki/Installing-wrk-on-OSX)


## Usage


### Basic

```
wrk -t12 -c400 -d30s http://localhost:8080/

```
12 threads

400 concurrent connections

30 seconds duration

### Pipeline

If you want to bump up the number of requests without using a lot of CPU resources, use pipeline. It duplicates the same request with a script. 

```
wrk -t4 -c128 -d30s http://localhost:8080 -s pipeline.lua --latency -- / 16
```

Here is the pipeline.lua script

```
init = function(args)
   request_uri = args[1]
   depth = tonumber(args[2]) or 1

   local r = {}
   for i=1,depth do
     r[i] = wrk.format(nil, request_uri)
   end
   req = table.concat(r)
end

request = function()
   return req
end
```

### Debug

If your wrk with pipeline.lua is not working or returning Non 2XX or 3XX code, chances are you are using the wrong request path after --. The default in the previous command is / and it should be changed to something like /v1/data for APIs.

In order to debug the pipeline.lua, use the following copy pipeline_debug.lua

```
init = function(args)
   request_uri = args[1]
   depth = tonumber(args[2]) or 1

   local r = {}
   for i=1,depth do
     r[i] = wrk.format(nil, request_uri)
   end
   req = table.concat(r)
end

request = function()
   return req
end

response = function (status, headers, body)
  if status ~= 200 then
    io.write("------------------------------\n")
    io.write("Response with status: ".. status .."\n")
    io.write("------------------------------\n")

    io.write("[response] Body:\n")
    io.write(body .. "\n")
  end
end
```

### Post

If you want to send a post request, you need to create a post.lua to define your post request. Here is an example.


```
wrk.method = "POST"
wrk.body = "{\"UmCreateAccountOpenReq\":{\"AccountOpenCreateCriteria\":{\"transitNumber\":\"12345\",\"accountServicedBy\":\"abc\"},\"HeaderReq\":{\"Header\":{\"sessionToken\":\"sessionToken1\",\"uiTransactionId\":\"uiTransactionId1\",\"language\":\"En\",\"uiContext\":\"uiContext\",\"version\":\"1\"}}}}"
wrk.headers["Content-Type"] = "application/json"
```

And here is the wrk command line that uses post.lua file. 

```
wrk -t2 -c100 -d30s -s post.lua http://localhost:8080/account/create
```

## Example

For some output examples, you can find at [benchmarks](https://github.com/networknt/microservices-framework-benchmark)

## Additional Info

There are two other [tutorial 1](https://www.digitalocean.com/community/tutorials/how-to-benchmark-http-latency-with-wrk-on-ubuntu-14-04)
and [tutorial 2](http://czerasz.com/2015/07/19/wrk-http-benchmarking-tool-example/) for wrk written by Michal Czeraszkiewicz

