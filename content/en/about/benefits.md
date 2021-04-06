---
title: Benefits of Light Platform
linktitle: Benefits
description: Improved performance, lowered production costs, security and ease of use are just a few of the reasons Light is so appealing.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
keywords: [productivity,cost,performance,security]
categories: [about,fundamentals]
menu:
  docs:
    parent: "about"
    weight: 4
weight: 4
sections_weight: 4
draft: false
aliases: []
toc: false
reviewed: true
---
## Lower the business requirement cost

Compared with the traditional software development process, the business analyst and designer can sit together to create an OpenAPI specification. It eliminates the need for a Word or Excel document which can never be traced and be update-to-date. Also, the spec can be understood by both computers and humans without any confusion or clarification. For client application requirements, it is much simpler as we just need to pick up services available in the organization or public to form an application, just like building with Lego.


## Lower the solution design cost

With opinionated frameworks and infrastructure, the solution design is much simpler. The output
would be an IDL document for services or services interactions for client applications. 


## Lower the development cost

With [light-codegen][] to generate a working project within a second, developers can focus on
the business logic implementation and unit test for the business logic. There is no need to handle
any cross-cutting concerns as they are taken care of by the platform with configurations. 

## Lower the devops cost

With devops as code, everything is automatic and there is no need for an engineer to build and
deploy services to a static shared environment anymore. 

## Lower the test cost

All tests will be done automatically to support CI/CD flow. Testers will write integration and end-to-end test cases and these tests will be part of the CI/CD flow to ensure the quality of the product. Also, all clients will submit their test cases to the service team to ensure that changes won’t break the client-side contract.

## Lower the production provision cost

With high throughput, low latency and a small memory footprint, the production provision cost will be significantly lower than with a traditional Java EE based solution. Based on different types of services, sometimes, the difference is over 100 times.

## Lower the operation cost

With centralized logging, metrics, auditing, tracing and monitoring, you will have peace
of mind during your production. 

## Lower application delivery cost

When building a new application, you don’t need to build every component. Instead, you just need to just pick the right building blocks from within the organization or the public. Building a new application is just like playing with Lego.

In summary, the goal of the platform is to lower the total cost of software products in all sizes of organizations. 



[light-codegen]: https://github.com/networknt/light-codegen
