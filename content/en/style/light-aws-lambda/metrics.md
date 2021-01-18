---
title: "Lambda Metrics"
date: 2020-12-10T10:22:06-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Metrics can be implemented as another extension. For each invoke to the Lambda function handler, we collect the metrics data. Some of the data elements are collected from the Enriched property from the custom authorizer. For example, the client_id from the JWT token and user_id from the JWT if authorization code flow is used.

In the logging design, we have mentioned that we need an HTTP server to be started in the same container as the Lambda function. The same server will be used to collect the metrics and send them to the control location. 

