---
title: "Debug Both Servers"
date: 2020-03-24T15:56:16-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

In the previous [debug with hybrid server][] tutorial, we have tried to put the user-query service in the hybrid-query server and use the JUnit test in the user-command service to debug the interaction between these two services. In this tutorial, we are going to run both user-query and user-command in servers within two IDE sessions. Instead of triggering the event with JUnit tests, we are going to use Postman to issue a request to either the query side or the command side.

### Environment

Please follow the [unit test tutorial][] to set up the local environment. 

### Long-lived Token

If you need a long-lived token for your test cases or command line, you can use the token in the [unit test tutorial][].

### Debug

In this session, we are going try the user registration and email confirmation from the Postman. 

Here is a vedio walkthrough. 

{{< youtube isiHPe3np9A >}}


[debug with hybrid server]: /tutorial/portal/developer/debug-hybrid-server/
[unit test tutorial]: /tutorial/portal/developer/debug-unit-test/
