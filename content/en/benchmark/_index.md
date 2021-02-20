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
reviewed: true
---

The light-4j framework is one of the fastest microservices/API frameworks on the market, and there is a lot of effort to make that happen. There are many frameworks and languages to build microservices, but the most popular ones are Java, Go and Nodejs.

Nodejs used to be faster than Java EE platforms, but it has many [issues][] in the enterprise world.

Go has potentials but is still not mature in enterprise applications. We are starting with Java because it is highly optimized, and there are so many existing investments in most organizations.

In Java, the most popular framework is Spring Boot, and a lot of users want to compare Light-4j with Spring Boot. We just started a new project [light-spring-boot][] to allow Spring Boot applications to leverage the Light-4j middleware handlers to address cross-cutting concerns with [better performance][]. 

We have tested most of the microservices frameworks, and here are the [benchmarks][] along with source code.

Also, we have submitted test cases to TechEmpower for the third-party test from [Round 14][] (Please note that light-4j was called light-java in this report) and [latest][] preview with some optimizations.

All the implementations for TechEmpower are following RESTful style, although many middleware handlers are removed. The source code can be used as an example of how to implement high-performance microservices.

* Microservices framework [benchmarks][] tested most popular microservices frameworks.
* Third-party [Techempower Benchmarks][] tested more HTTP frameworks with contributions open source community.
* Leverage Light-4j middleware for Spring Boot App with [light-spring-boot][] for [better performance][].
* Light-proxy sidecar for CCC benchmarks with `wrk` for [maximum throughput][] with both Light-4j and Spring Boot backends.

[benchmarks]: https://github.com/networknt/microservices-framework-benchmark
[issues]: /benchmark/nodejs/
[round 14]: https://www.techempower.com/benchmarks/
[latest]: https://www.techempower.com/benchmarks/#section=test&runid=58042695-831a-4a35-8c60-2b872a06f799&hw=ph&test=json
[light-spring-boot]: https://github.com/networknt/light-spring-boot
[better performance]: /benchmark/spring-boot/
[maximum throughput]: /service/proxy/benchmark/
