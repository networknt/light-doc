---
title: "Proxy Artifact"
date: 2017-12-16T09:44:42-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When considering using light-proxy to wrap your existing API, you have two options for deployment artifacts. With each of the following options, you have to provide config files and pass them as a directory to the light-proxy.


### Docker

The docker image can be found at https://hub.docker.com/repository/docker/networknt/light-proxy/tags

```
docker pull networknt/light-proxy:latest
```

### Fatjar

You can download the fat jar from maven central with all the dependencies included.

### Source Code

It makes sense for enterprise users to fork the source code repository and add their customized handlers to make the light-proxy suitable to be the sidecar container in front of all the back-end APIs to handle cross-cutting concerns on the network level. It will allow the API developers to focus on business logic only with different frameworks in different languages and still join the light platform's service mesh along with light-router and light-portal. 







