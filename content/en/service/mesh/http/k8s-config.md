---
title: "Kubernetes Config"
date: 2021-07-29T17:50:18-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

When using the http-sidecar within a Kubernetes cluster, there are some configuration considerations: 

### logback.xml

This is the file used to set up the logging level on the http-sidecar and it should have a copy per environment in the overlays folder with the following reasons. 

* Although the http-sidecar has the logger config endpoints to change the logging level from the control plane, we usually pass through the request to the backend API in the same pod. It is more useful to update the backend API's logging level to diagnose production issues in most cases. 

* Sometimes, we need the debug log during the http-sidecar startup to figure out connectivity issues with the config server. It is before the control plane is available to manipulate the debug level on the sidecar.

