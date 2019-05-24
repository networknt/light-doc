---
title: "Router Artifact"
date: 2018-02-12T14:33:15-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When considering using light-router to assist the client that cannot leverage client.jar to access APIs deployed in the cloud you have two options for deployment artifacts. With each of the following option, you have to provide config files and pass them as a directory to the light-router or as Kubernetes Secret to the container. For more information on how to config the light-router, please refer to [configuration][].

### Docker

The docker images can be found at https://hub.docker.com/u/networknt/dashboard/

### Fatjar

The fatjar can be downloaded from maven central with all the dependencies included.


[configuration]: /service/router/configuration/
