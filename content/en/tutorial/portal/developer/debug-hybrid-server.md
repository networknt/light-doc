---
title: "Debug Hybrid Server"
date: 2020-03-21T10:35:29-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When debugging a portal service, you can use a JUnit test case from the IDE if the service endpoint is working independently. However, if the endpoint needs to access the query side for additional information, you need to run the query side in a hybrid server to support the command side debugging in IDE. Of course, you can still debug the query side service in a hybrid server to see the interaction between services. 

### Environment

Please follow the [previous tutorial][] to set up the local environment. 

### Long-lived Token

If you need a long-lived token for your test cases or command line, you can use the token in the [previous tutorial][].

### Debug

In this tutorial, we are going to leverage user-command and user-query services in the light-portal; however, the concept works for other services as well. When testing an endpoint from the user-command service with a JUnit test case, the endpoint will need to access one of the endpoints from the user-query service. Here we show users how to set up a debug session for hybrid-query server and load the user-query service in IDE in debug mode. 

Here is a vedio walkthrough. 

{{< youtube GNjp_HHSV4E >}}

[previous tutorial]: /tutorial/portal/developer/debug-unit-test/

