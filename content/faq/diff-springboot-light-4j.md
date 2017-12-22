---
title: "Difference between SpringBoot and Light-4j"
date: 2017-12-21T06:34:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The major difference is Spring Boot is based on servlet container and it is blocking
and light-4j is based on non-blocking Undertow core. That is why there is huge gap
in performance. 

As this [benchmark][] shows one light-4j instance can complete with over 100's Spring 
Boot instances for the same throughput and latency not even counting it uses 5 times
more memory per instance.

Another big difference is developer productivity increase as light-4j generates the
code from specification, IDL or schema and allow developers only working on the
business handler and business handler test cases without even knowing that all other
cross-cutting concerns. For Spring, developers have to taking care of configurations
and annotations.

[Spring framework][] was designed to compete with [Java EE][] and it has to provide
all the features of Java EE and it is as heavy as Java EE. It is not possible to
build light-weight cloud application without eating too much resources.

[benchmark]: https://github.com/networknt/microservices-framework-benchmark
[Spring framework]: /architecture/spring-is-bloated/
[Java EE]: /architecture/jee-is-dead/
