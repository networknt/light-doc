---
title: "Deploy Patterns"
date: 2021-06-02T15:39:43-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The sidecar approach has a separate backend API process compared with the embedded cross-cutting concerns in the same process. We can deploy the two processes in the same container or the sidecar process in a different container in a pod. 

Putting the sidecar and backend API in the same container can reduce the resource usage overall, as an additional container will have its OS. However, there are several benefits if the sidecar is in its own container.


### Development Flexibility

With a separate container, developers can have their base image on what makes sense for the backend service, not what is required for the HTTP Sidecar. 

When the sidecar and the backend API share the same container, we have to dynamically build the container image with multiple layers to meet both sidecar and backend API requirements. It increases the build time for developers for each update, build and test cycle. 

With a separate container for the sidecar, developers can start the sidecar in a container on their desktop and run the backend API in their IDE to debug locally.

A separate container for the sidecar also gives us the opportunity to work with legacy APIs running on VMs or Lambda Functions running on AWS. 

### Deployment Flexibility

Even when we have two containers, they will be deployed in the same pod in a Kubernetes cluster. 

It is hard to standardize the base image when two processes share the same container as different languages have their recommended base image from the vendor. 

There might be environment variables passed into the container, and there would be conflicts in the naming of the variables between the sidecar and backend API. 

With a separate container, it is isolated from the backend service and can be configured independently.

Monitoring and issue resolution will be simpler with separate containers.

### Maintannce Flexibility

Change of the sidecar or backend API can be deployed independently without impacting each other.


### Summary

Given the above analysis, we recommend that the sidecar be deployed with a separate container in the same pod as the backend API. The concern of extra latency is negligible based on this performance test [report](/service/proxy/benchmark/). The extra provisioning cost is justified with the flexibility for a separate sidecar container. 

