---
title: "Benchmark Overview"
date: 2017-11-06T11:16:05-05:00
description: "Benchmark of Light Platform"
categories: []
keywords: [benchmark]
menu:
  docs:
    parent: "benchmark"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Light-4j framework is one of the fastest API frameworks on the market and there are
a lot of effort to make that happen. There are a lot of frameworks, languages to build
microservices but most popular ones are Java, Go and Nodejs.

Nodejs used to be really fast compare with Java EE platforms but it has a lot of
[issues][] in enterprise world.

Go has potentials but is still not mature in enterprise applications. We are starting
with Java due to the fact that it is highly optimized and there are so many existing
investments in most of organizations.

We have tested most of microservices frameworks and here is the [benchmarks][] along
with source code.

Also, we have submitted test cases to TechEmpower for third party test from [Round 14][]
(Please note that light-4j was called light-java in this report) and [Round 15][]
preview with some optimizations.

All the implementations for TechEmpower are following RESTful style although there are 
a lot of middleware handlers are removed. The source code can be used as an example on 
how to implement high performance microservices.




[benchmarks]: https://github.com/networknt/microservices-framework-benchmark
[issues]: /benchmark/nodejs/
[round 14]: https://www.techempower.com/benchmarks/
[round 15]: https://www.techempower.com/benchmarks/previews/round15/