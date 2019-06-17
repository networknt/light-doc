---
title: "Test light service"
date: 2017-11-06T17:45:44-05:00
description: ""
categories: []
keywords: [tutorial]
aliases: []
toc: false
draft: false
---

CI/CD pipeline needs different type of tests to ensure the quality of the product before 
deploy on production. In microserivces architecture, the integration test is different
than monolithic application as we need to have all upstream and downstream services to be
up and running at the same time to test the interaction of these services. There are four
type of tests we need to perform before release the service to production - unit test, 
integration test, consumer contract test and end-to-end test. 

* [Unit Test](/tutorial/common/test/unit-test/)
* [Integration Test](/tutorial/common/test/integration-test/)
* [End-to-End Test](/tutorial/common/test/end-to-end-test/)
* [Consumer Driven Contract](/tutorial/common/test/consumer-driven-contract/)
