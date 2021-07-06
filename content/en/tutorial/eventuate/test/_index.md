---
title: "Test Tutorial"
date: 2018-01-06T20:56:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

As there are several dependencies for light-eventuate-4j applications to run, it is a little
complicated to test services built on top of light-eventuate-4j compare with other simple
services. 

In this section, we will walk through the Unit Test, Integration Test and End-to-End Test with
real examples so that others can learn what is the difference of each type of test and how
to write these test ensure high quality services are produced. 

[Unit Test][] is located in /test folder in src and these test cases will have no external dependencies.

[Integration Test][] is located in /integration folder in src and these test cases will have external dependencies.

[End-to-End Test][] is normally locate in light-bot and it will run in an environment that all related services will be running at the same time.


[Unit Test]: /tutorial/eventuate/test/unit-test/
[Integration Test]: /tutorial/eventuate/test/integration-test/
[End-to-End Test]: /tutorial/eventuate/test/end-to-end-test/
