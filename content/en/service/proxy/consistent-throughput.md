---
title: "Proxy Benchmark with Consistent Throughput"
date: 2021-02-17T11:25:15-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In the [benchmark with maximum throughput][], our focus is the max throughput with `wrk`; however, most customers won't need millions of requests per second. Some users are more interested in the performance of light-proxy with consistent volume. 


The `wrk` is not the tool to generate consistent volumes, so we have to find another tool. We will try wrk2 and JMeter for the tests to cross-verify the results from two different tools. 

### wrk2

We will use wrk2 docker for the test and the command line is very similiar with the wrk. Here is the command line with Docker. 

```
docker run --rm --net=host 1vlad/wrk2-docker -t1 -c1 -d30s -R50 --latency https://localhost:8443/v1/pets
```

The above command will limit the number of requests to 50 per second.


### Light-4j Standalone

We start both the light-4j backend and the proxy together with the docker-compose and connect to the backend in this test.

```
cd ~/networknt/light-example-4j/proxy
docker-compose -f docker-compose-light-4j.yml up
```

##### R50

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   797.32us  427.16us   2.09ms   66.73%
    Req/Sec    26.06     73.48   666.00     88.70%
  1536 requests in 30.32s, 289.50KB read
  Socket errors: connect 0, read 0, write 0, timeout 507
Requests/sec:     50.65
Transfer/sec:      9.55KB

```


##### R100

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   766.63us  409.23us   2.06ms   64.29%
    Req/Sec    52.86     95.06   555.00     77.53%
  3072 requests in 30.32s, 579.00KB read
Requests/sec:    101.31
Transfer/sec:     19.09KB

```

##### R500

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   781.77us  431.76us   6.33ms   67.20%
    Req/Sec   263.70    113.00   600.00     68.96%
  14992 requests in 30.00s, 2.76MB read
Requests/sec:    499.69
Transfer/sec:     94.18KB
```

##### R1000


```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   812.61us  408.66us   4.93ms   67.68%
    Req/Sec   537.27    180.01     1.33k    85.65%
  29905 requests in 30.00s, 5.50MB read
Requests/sec:    996.82
Transfer/sec:    187.88KB

```

##### R5000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.89ms  424.30us   5.01ms   65.58%
    Req/Sec     2.63k   446.07     3.78k    61.90%
  149260 requests in 30.00s, 27.47MB read
Requests/sec:   4975.08
Transfer/sec:      0.92MB

```

##### R10000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.89ms  412.57us  12.19ms   65.26%
    Req/Sec     5.32k   381.55     7.89k    69.64%
  298473 requests in 30.00s, 54.94MB read
Requests/sec:   9948.88
Transfer/sec:      1.83MB

```


### Light-4j Proxy

##### R50


```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.03ms  376.06us   2.16ms   67.64%
    Req/Sec    26.64     75.86   666.00     88.24%
  1536 requests in 30.33s, 289.50KB read
  Socket errors: connect 0, read 0, write 0, timeout 508
Requests/sec:     50.64
Transfer/sec:      9.55KB

```

##### R100

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.02ms  391.64us   2.81ms   68.90%
    Req/Sec    54.01    110.73     1.00k    78.68%
  3072 requests in 30.32s, 579.00KB read
Requests/sec:    101.31
Transfer/sec:     19.09KB

```

##### R500

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.02ms  405.37us   2.69ms   66.10%
    Req/Sec   265.02    114.60   666.00     68.21%
  14992 requests in 30.00s, 2.76MB read
Requests/sec:    499.69
Transfer/sec:     94.18KB

```

##### R1000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.00ms  413.89us   3.17ms   66.09%
    Req/Sec   527.27    193.22     1.33k    85.73%
  29901 requests in 30.00s, 5.50MB read
Requests/sec:    996.59
Transfer/sec:    187.83KB

```

##### R5000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.07ms  474.74us  13.26ms   66.23%
    Req/Sec     2.62k   241.07     4.00k    66.68%
  149268 requests in 30.00s, 27.47MB read
Requests/sec:   4975.45
Transfer/sec:      0.92MB

```

##### R10000

```

docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.14ms  629.33us  21.39ms   78.93%
    Req/Sec     5.28k   433.56     8.50k    73.55%
  298454 requests in 30.00s, 54.93MB read
Requests/sec:   9948.54
Transfer/sec:      1.83MB

```


### Spring Boot Standalone

##### R50


```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.95ms  398.95us   3.49ms   65.93%
    Req/Sec    26.67     77.75   777.00     88.64%
  1536 requests in 30.33s, 319.50KB read
  Socket errors: connect 0, read 0, write 0, timeout 507
Requests/sec:     50.65
Transfer/sec:     10.54KB

```

##### R100

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.98ms  438.02us   4.22ms   70.78%
    Req/Sec    54.05    106.01   777.00     78.28%
  3072 requests in 30.32s, 639.00KB read
Requests/sec:    101.31
Transfer/sec:     21.07KB

```

##### R500

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.22ms  661.78us   5.51ms   72.50%
    Req/Sec   261.80    809.93     6.67k    97.66%
  14798 requests in 30.19s, 3.01MB read
Requests/sec:    490.23
Transfer/sec:    102.05KB

```

##### R1000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.94ms  436.64us   5.12ms   67.43%
    Req/Sec   522.83    195.34     1.33k    90.03%
  29901 requests in 30.00s, 6.08MB read
Requests/sec:    996.59
Transfer/sec:    207.46KB

```

##### R5000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8080/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.04ms  489.36us   6.78ms   69.48%
    Req/Sec     2.66k   326.45     4.11k    63.89%
  149257 requests in 30.00s, 30.35MB read
Requests/sec:   4975.22
Transfer/sec:      1.01MB

```

##### R10000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8080/v1/pets
```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8080/v1/pets
Running 30s test @ http://192.168.1.144:8080/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.24ms  655.98us   7.34ms   73.45%
    Req/Sec     5.26k     1.34k    7.90k    75.39%
  294238 requests in 30.00s, 59.82MB read
Requests/sec:   9806.85
Transfer/sec:      1.99MB

```


### Spring Boot Proxy

##### R50


```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R50 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.22ms  419.07us   4.00ms   67.04%
    Req/Sec    25.67     77.01   800.00     88.37%
  1536 requests in 30.33s, 339.26KB read
  Socket errors: connect 0, read 0, write 0, timeout 507
Requests/sec:     50.65
Transfer/sec:     11.19KB

```

##### R100

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R100 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.18ms  409.40us   4.64ms   67.81%
    Req/Sec    52.35     96.34   666.00     77.46%
  3072 requests in 30.33s, 678.24KB read
Requests/sec:    101.30
Transfer/sec:     22.37KB

```

##### R500

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R500 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.65ms    0.87ms  10.67ms   78.19%
    Req/Sec   260.19    803.08     6.67k    97.58%
  14792 requests in 30.19s, 3.19MB read
Requests/sec:    490.01
Transfer/sec:    108.23KB

```

##### R1000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R1000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.61ms  848.86us  10.25ms   77.09%
    Req/Sec   528.10      1.14k    6.67k    95.66%
  29483 requests in 30.06s, 6.36MB read
Requests/sec:    980.87
Transfer/sec:    216.67KB

```

##### R5000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R5000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.32ms  623.80us  11.24ms   75.06%
    Req/Sec     2.63k   305.12     4.67k    71.05%
  149266 requests in 30.00s, 32.20MB read
Requests/sec:   4975.38
Transfer/sec:      1.07MB
```

##### R10000

```
docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8000/v1/pets

```

Result:

```
steve@freedom:~$ docker run --rm --net=host 1vlad/wrk2-docker -t2 -c128 -d30s -R10000 http://192.168.1.144:8000/v1/pets
Running 30s test @ http://192.168.1.144:8000/v1/pets
  2 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.66ms    1.13ms  18.64ms   87.51%
    Req/Sec     5.27k   787.79    10.78k    78.84%
  298423 requests in 30.00s, 64.37MB read
Requests/sec:   9947.15
Transfer/sec:      2.15MB
```


### Summary



### Conclusion



[benchmark with maximum throughput]: /service/proxy/maximum-throughput/

