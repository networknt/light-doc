---
title: "All in One"
date: 2019-06-15T20:36:28-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Your applications must be instrumented before they can send tracing data to Jaeger backend. Before that, we need to start the tracer. 

All-in-one is an executable designed for quick local testing, launches the Jaeger UI, collector, query, and agent, with an in-memory storage component.

The simplest way to start the all-in-one is to use the pre-built image published to DockerHub (a single command line).

```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.9
```

You can then navigate to `http://localhost:16686` to access the Jaeger UI. There should only be one service on the UI called jaeger-query now. 

Please be aware that this is only for local testing. For production, please visit the following links. 

* Kubernetes templates: https://github.com/jaegertracing/jaeger-kubernetes
* Kubernetes Operator: https://github.com/jaegertracing/jaeger-operator
* OpenShift templates: https://github.com/jaegertracing/jaeger-openshift

Now we have all the component of the tracer started. In the next step, we are going to reuse the service discovery application we built before to add additional steps for distributed tracing. 

Before we dive into a distributed application, let's first modify the openapi [petstore](https://github.com/networknt/light-example-4j/tree/release/rest/openapi/petstore) application to enable tracing and get a feeling of it. 

