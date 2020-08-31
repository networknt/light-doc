---
title: "Rest With Database Access"
date: 2020-08-30T23:38:17-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Light-4j uses a light-weight HTTP core that can serve millions of requests per second. None of the existing SQL databases can have the throughput to match it. 

When using the SQL database, we usually use an in-memory cache to serve the data faster. Or we go with Kafka streams for the local RocksDB store for key/value access. 

For direct database access, please refer to the examples in [techempower repo](https://github.com/TechEmpower/FrameworkBenchmarks/tree/master/frameworks/Java/light-java). 

