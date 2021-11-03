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

### client.truststore

The http-sidecar will connect to the OAuth 2.0 provider to get the JWK to verify the JWT token. Also, it will connect to the config server to get the configuration files and connect to the control plane to register itself and discover other downstream services. All above accesses need a TLS connection, and a certificate needs to be added to the client.truststore that is referred in client.yml by the light-4j Http2Client module. To ensure environment segregation, we should create a client.truststore per overlay so that dev http-sidecar can only connect to dev OAuth 2.0 provider, dev config server and dev control plane even if the dev and sit are deployed in the same Kubernetes cluster.


