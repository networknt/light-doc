---
title: "SideCar"
date: 2020-11-24T10:25:43-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

One of the main features for light-4j frameworks is separating the cross-cutting concerns from the business logic so that developers can focus on the business API handler implementations and DevOps can focus on the cross-cutting concerns in the pipeline for deployment. 

All cross-cutting concerns are implemented as middleware handlers that can be plugged into the HTTP request/response chain. All of them group together to form a flexible gateway to protect the API handler implementations. 

These plugins can be embedded in the same process in the API implementation as an embedded gateway. However, this might limit API implementation on the light-4j frameworks. 

![embedded](/images/embedded-chain.png)


To support other frameworks and languages, we can deploy the light-proxy container as a sidecar along with API implementation. 


![proxy](/images/sidecar-chain.png)
