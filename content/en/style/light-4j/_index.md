---
title: "Light-4j for absolute performance"
date: 2019-03-10T21:39:28-04:00
description: ""
categories: [style]
keywords: [light-4j]
menu:
  docs:
    parent: "style"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Light-4j is the main component in the light platform. It contains a client, a server and all common HTTP cross-cutting concerns implemented in the request/response chain as middleware handlers. An embedded gateway is established in the same server process by injecting these middleware handlers between the client and server in the HTTP channel via configuration in the DevOps process to gain higher performance and lower memory footprint. This allows developers to focus on business logic only to achieve higher productivity. 

Most users won't use light-4j directly to build their API but choose one of the API styles. For each style, we have additional middleware handlers to address style-specific protocol details. 


* [Concerns Overview](/concern/)
* [GitHub Repository](https://github.com/networknt/light-4j)
* [TechEmpower Benchmarks](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=plaintext)

