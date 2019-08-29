---
title: "Peformance"
date: 2019-08-28T23:21:49-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-4j framework is very light and supports millions of request per seconds on a commodity server. To show our users how to build high-performance microservices, We have submitted benchmark implementations to Techempower to compare light-4j with other frameworks on the market. The source code can be found in [techempower repo][], and it is an excellent example of how to build high-performance services, including database access. The benchmark result can be found at [techempower benchmark][] 

We later found that most so-called frameworks are just hundreds of line of customized code optimized against the TechEmpower testing environment. So we have created our benchmark, which is microservices focused. The [benchmark repository][] is in networknt organization, and it compares all popular microservices frameworks on the market. The test is done on a desktop computer and focuses on the raw throughput and latency of each framework. There might be more test cases added to the suite in the future. Users can easily clone the repository and build the frameworks they are interested in and perform tests on their own. 


[techempower repo]: https://github.com/TechEmpower/FrameworkBenchmarks/tree/master/frameworks/Java/light-java
[techempower benchmark]: https://www.techempower.com/benchmarks/
[benchmark repository]: https://github.com/networknt/microservices-framework-benchmark
