---
title: Benefits of Light Platform
linktitle: Benefits
description: Improved performance, Lowered production cost, security and ease of use are just a few of the reasons Light Platform is so appealing.
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2017-02-01
keywords: [productivity,cost,performance,security]
categories: [about,fundamentals]
menu:
  docs:
    parent: "about"
    weight: 30
weight: 30
sections_weight: 30
draft: false
aliases: []
toc: false
---
## Lower the business requirement cost

Compare with the traditional software development process, the business analysis and designer
can sit together to create an OpenAPI specification. It eliminate the need for Word or Excel
document which can never be traced and update-to-date. Also the spec can be understood by both
computers and humans without any confusion or clarification. For client application requirement
it is much simpler as we just need to pickup services available in the organization or public
to form an application just like building with Lego.

## Lower the solution design cost

With opinionated frameworks and infrastructure, the solution design is much simpler. The output
would be an IDL document for services or services interactions for client applications. 


## Lower the development cost

With [light-codegen][] to generate a working project within a second, developers can only focus on
the business logic implementation and unit test for the business logic. There is no need to handle
any cross-cutting concerns as they are taken care of by the platform with configurations. 

## Lower the devops cost

With devops as code, everything is automatic and there is no need for engineer to build and
deploy services to static shared environment anymore. 

## Lower the test cost

All tests will be done automatically to support CI/CD flow. Testers will write integration and
end-to-end test cases and these tests will be part of the CI/CD flow to ensure the quality of
the product. Also, all clients will submit their test cases to the service team to ensure that
changes won't beak the client side contract. 

## Lower the production provision cost

With high throughput, low latency and small memory footprint, the production provision cost
will be significant lower than traditional Java EE based solution. Based on different type
of services, sometime, the difference is over 100 times. 

## Lower the operation cost

With centralized logging, metrics, auditing, tracing and monitoring, you will have the peace
of mind on your production. 

## Lower application delivery cost

When building a new application, you don't need to build every component but just pick the
right building blocks from within the organization or public. Building a new application is
just like playing Lego games. 
 
In summary, the goal of the platform is to lower the total cost of software product in
all sizes of organizations.  



[light-codegen]: https://github.com/networknt/light-codegen
