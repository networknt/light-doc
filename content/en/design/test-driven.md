---
title: "Test Driven"
date: 2018-04-22T08:37:11-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When an organization adopts microservices architecture, CI/CD pipeline is essential to ensure the quality of the delivery. Unlike the common practices for monolithic application to do the integration test after the entire application is completed, we need continuous integration from day one. 

When using light-platform to build microservices, two use cases need continuous integration: 

* Often a team of developers focuses on middleware handlers that are shared by numeric microservices. These developers need to do integration test with microservices built by other teams to ensure that any changes in these middleware handlers will not break any microservices. 

* When breaking a big monolithic application to smaller pieces, one developer might be responsible for one or more services. If there are dependencies between services, these services need to be tested together although they can be developed and deployed independently. There is no need to wait until all services are completed. The integration test should be started on day one and integrated one endpoint each time.

There are four different testing levels supported by the platform. You can also find [tutorials][] to learn how to use these type of tests in light-platform. 
 

### Unit Test

Unit test cases are designed to test utility classes or classes called from business handlers. They need to be created by the use manually while developing the application or service. These test cases should run fast as they will be executed every time a developer is calling mvn package. 

### Integration Test

Integration test cases rely on the target server to be up and running to hit the live server to perform the tests. When light-codegen scaffold a new project, it includes one integration test case per endpoint. Also, a test server is generated to start the server with all the default configurations except server.yml to listen to a different port.  You should write as many test cases as possible to test all positive and negative scenarios. 

### Consumer Contract Test

Consumer contract testing is not new, but with the mainstream acceptance of microservices, people realize that consumer-driven contracts are an essential part of a mature microservice testing portfolio. However, we want to point out that consumer-driven contract testing is a technique and an attitude that requires no special tool to implement. Frameworks like Pact makes proper contract tests easier to implement; However, we should not focus on the framework but the general practice. Writing Pact tests is not a guarantee that you are creating consumer-driven contracts; likewise, in many situations, you should be creating good consumer-driven contracts even where no pre-built testing tool exists.

### End-to-End Test

End-to-End testing involves all depending services and upstream/downstream services. They need to be tested together to ensure all the services can work in concert. It is used to be very hard to do, but with Docker and cloud environment, it is relatively easy to perform with proper tools. We found that all existing DevOps tools are too monolithic focused and they are not suitable to design pipeline for microservices which have multiple repositories. This is why we have built [light-bot][] for building and end-to-end testing. 


[light-bot]: /tool/light-bot/
[tutorials]: /tutorial/common/test/