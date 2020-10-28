---
title: "Development Flow"
date: 2017-11-06T17:18:18-05:00
description: ""
categories: []
keywords: []
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

### Three Streams

Three flows are running in parallel but not started at the same time in API development.

The data architect and business analyst start API Specification first.

API developers start API implementation after the first release of the specification.

Client implementations start almost simultaneously as the API implementation team for mock API can be generated from OpenAPI specification immediately. An API in a chain of call stacks can be considered as both a server and a client.

The specification might be changed during the process, and the changes will be propagated to the API implementation team and Client implementation team(s). 

![API Flow](/images/api_flow.png)


### Shift Left

Unlike monolithic, microservices architecture can scale in development. By dividing a big application into smaller pieces, we can put more developers into the team. Everybody is working against specifications that can glue each piece together at the integration test. It dramatically increases productivity in the development cycle. 

With some tools and procedures, we can integrate the DevOps flow to the developers' laptop to shift everything to the left. It is more cost-effective to reveal and fix defects and other issues earlier than later, which helps remove code instabilities and improve quality. 

Here we use activities on a developer's laptop to describe the flow in different life-cycle stages. Let's assume service A calls service B and service B calls service C. A developer is working on service B at the moment. Each service has ten endpoints, and each team can complete one endpoint per day. 

Instead of waiting for all three services to complete to start the integration test on day ten, we start the integration test on day one when each service has only one endpoint completed. At the end of day two, we have the second endpoint integrated and ensure the first one is not broken. 

Not only can we shift the integration between services left, but we can also integrate the cross-cutting concerns as earlier as possible. Whether you embedded cross-cutting into the request/response chain in the same service process or use light-proxy/light-router to apply the cross-cutting concerns on the network level with sidecars, we can enable the CCC on the developer's laptop during the integration test. 

### Developer Flow

##### Business Handler

Separating the cross-cutting concerns from business logic allows developers to focus on business handlers during the development. What they need to think about in the beginning is how to generate a response given a request. At this stage, all the work can be done within the IDE. Write the handlers and unit test cases and test them within the IDE. Sometimes, you might leverage some online services. If a database is needed, it can be started using docker on the laptop or connecting to a shared instance on the network. At this stage, the focus is only on the business handles. For junior and some intermediate developers, this might be all the work they need to do. 

##### Cross-Cutting Concerns

Once the developer is comfortable with his/her handlers, he/she can bring the cross-cutting concerns into the picture. It means to enable embedded cross-cutting concerns middleware handlers via configuration changes within the same instance or start a light-proxy instance in a docker container or simple a Java process within another IDE for debugging. At this moment, we can bring the light-bot DevOps tool to do it automatically.

Testing at the stage will be based on the OpenAPI specification, and the tests will be covering the API as a whole. Other developers can use your repository or docker images for their integration tests. 

##### Mock Downstream API

If your API depends on one or more downstream APIs that are not ready, you can generate these APIs with their specifications. By leveraging the light-codegen, you can scaffold a project and mock the handlers to return something you are expecting from the downstream API temporarily. Once the downstream APIs are ready, you can replace the mock API with the real ones.

##### Manual Integration

Once a downstream API is ready to be integrated, we can build locally from the source repository or pull it from the docker repository. As this is the first time integration for the endpoint, it might be better to start both in IDE to debug across two or more APIs. Once the integration is done, we can put it into the light-bot config to run it automatically. 

##### Automatic Integration

In this step, we are going to leverage light-bot to run the integration tests automatically. We can ensure our API can call downstream APIs successfully, at least for some endpoints, in the above step. We need to configure the light-bot to check out all integrated APIs, start them all together along with sidecars, test against defined requests, and verify the response associated with the requests. 

It can be done as standalone processes or docker-compose or docker-swarm or Kubernetes cluster on the local laptop. The integration test should be executed several times each day against source repositories to ensure new changes won't break anything. 

##### DIT Testing

Once we reach a milestone, several services can be brought to the DIT cluster to test them all together. At this time, we can run the integration test cases from different API developers together. We need to take an extra step to merge the integrationt test from different teams in the light-bot configuration. Along with the integration test, we can enable the non-functional tests in this step. It allows us to find out performance issues and resolve them as early as possible.


##### SIT Testing

This is an optional step that controlled by our delivery team. Here we can perform all tests in an official environment. The real client application might be involved and certain manual tests might be executed to verify the integratiy of the application as a whole. Bofore any manual testers involved, we can run the automatic tests to ensure the functionality and connectivity. 

##### UAT

UAT might performed by another team and it can be automatic or manual. 

##### PROD

Production release is the end of the current cycle and the beginning of the next iteration. 

