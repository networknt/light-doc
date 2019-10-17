---
title: "Microservices Architecture"
date: 2017-11-06T11:14:26-05:00
description: ""
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

Microservice architecture, or simply microservices, is a distinctive method of developing software that has grown in popularity in recent years. In fact, even though there is not a whole lot out there on what it is and how to do it, for many developers, it has become a preferred way of creating enterprise applications. Thanks to its scalability, this architectural method is considered particularly ideal when you have to enable support for a range of platforms and devices—spanning web, mobile, Internet of Things, and wearables—or simply when you are not sure what kind of devices you will need to support in an increasingly cloudy future.

While there is no standard, formal definition of microservices, there are certain characteristics that help us identify the style. Essentially, microservice architecture is a method of developing software applications as a suite of independently deployable, small, modular services in which each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal.

Sam Newman defines microservices as "Small Autonomous, independently releasable services that work together, modeled around a business domain" in his book.  

To begin to understand microservices architecture, it helps to consider its opposite: the monolithic architectural style. Unlike microservices, a monolith application is always built as a single, autonomous unit. In a client-server model, the server-side application is a monolith that handles the HTTP requests, executes logic, and retrieves/updates the data in the underlying database. The problem with a monolithic architecture, though, is that all change cycles usually end up being tied to one another. A modification made to a small section of an application might require building and deploying an entirely new version. If you need to scale specific functions of an application, you may have to scale the entire application instead of just the desired components. This is where creating microservices can come to the rescue.  

## Using Conway’s Law for fast development

Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations — Melvin E. Conway

Conway’s Law was first stated in 1967, and it has been proven empirically again and again. To understand it, let’s discuss a few examples of its application:

* Teams who can communicate easily (typically small and collocated) tend to create monoliths

* Teams who communicate through intermediaries (managers, architects, etc.) tend to work on separate modules

* Teams who communicate through hierarchies tend to work on complex structures. For example, one core/framework team, front-end teams, back-end teams, etc.

Fred George started one of the first microservices projects with a big challenge: the work had to be organized in a way that allowed adding developers fast. He decided to use Conway’s Law to their advantage, and create an architectural structure that allowed adding pairs of programmers to the development. The only option was to create many small services that communicated asynchronously and that were completely independent of each other, each developed by a pair of programmers. 
 

## Principles of microservices

We often use a set of principles to guide decision making, and the following is a list of microservices principles. 

#### Modelled Around Business Domain

Unlike monolithic application layered with presentation, business logic and repository which is cut horizontally. Micorservices encourage breaking monolithic application to each function domain which is cut vertically. In SOA monolith, the application is divided by technical layers and in MSA,
the application is divided by business bounded context. 

If a system is divided in in technical layers, then every small change for example adding a new field will involve three teams to update UI, Business Logic and Data Repo. The scenario will be getting worse if the application is getting bigger as the interface between layers are not stable and subject to
change due to influx of business requirements. 

By decomposing services around business domain can make the interface relative stable as it represent business function which is more stable then technologies domain. Also, each team will have knowledge on the particular business domain and responsible to support it as well. 

The book "Domain-Driven Design" by Eric Evans is a very good one that can guide you to model microservices. It was published over a decade ago but only getting popular recently as architecture shift and tool chain is ready for it.    

#### Culture of Automation

If you only deploy several services, then manual VM provision and deploy your services is doable. However, if you have thousands of services, the entire process must be automatic to allow this kind of scale. When we think about automation, we are thinking about infrastructure, platform and tool chains. This will make the microserivces architecture very expensive for the front investment. Normally, for an organization to adopt microservices architecture, it take them serveral months to deploy 2 or 3 services and then they start to realize that they need to invest the ecosystem to support services and working on it for several months. Once the platform is ready, the number services deployed will be exploded in a couple of months.

We need to automated testing at four different levels: unit tests, consumer driven tests, integration tests and end-to-end test. These tests can give us confidence to delivery to production on the same day code is changed. 

We need to have continuous delivery pipeline to deploy the service once the code is merged to master branch.


#### Hide Implementation Details

Microservice should hide its implementation details and the only interface to the outside is the interface contract. More often, people want to share the same database between multiple services and this is a very bad design borrowed from monolith world. This design make these services tightly coupled at db level and it is very hard for one service to add a new field into a table or change the name of a column name.

Every service should have its own database and this allow them to choose the right db for the job and they can switch db if it is necessary. Also they can change their implementation language or framework if new technologies emerged. 

Another perspective is about interface design around bounded contexts and aggregates. This will guarantee that the exposed interface is design for the business domain without revealing details. 

#### Decentralize All The Things

Micorserivces architecture is optimized around autonomy and each team is responsible for one to many serivces. They are given as much freedom as possible to do the job at hand. Do they need to open a ticket to create a VM to deploy a test instance or they can do that on their own? Do they have shared
governance so that teams can work together to make decision without central authority?  

It also comes with technical decisions. Are we going to use a centralized API gateway in front of all services? Do we need an EMB to do some magic in order to make services work together? Or we adopt dumb-pipes, smart endpoints to have a message broker layer for service to service communication without
business logic?


#### Deployed Independently

You deploy your service to production without impacting other services. If you need to deploy your service because other services are deploying, something is wrong and it needs to be fixed. From operation point of view, we need to figure out if we want to make every service totally isolated or several services can share the same host/vm. A new service deployed to a shared host might hog the CPUs and brought down all other services on the same host. 

But how can I ensure that my service deploys to production won't impact others? Most people will say that we have all related services deployed and test them end-to-end but in reality, this might not be possible or very costly. As services are designed with an interface contract, [consumer-driven contract][]
testing is here for rescue. All consumers will have their expectations built as test cases and submitted to service provider. These tests must be passed in order to release to production and it also help the service provider to identify which consumer is broken if a change is occurring.  

Sometimes you have to introduce breaking upgrade on your service and you can have two versions of the service instances running in parallel for a while until all consumers upgrade to the latest version. 


#### Consumer First

Serivces exist to be called from the interface provided. When designing APIs, you need to involve the consumers to understand their needs and deigned the contract based on the consumer requirement as long with your understanding of the domain. 

Before services are built, the IDL(interface definition language) must be completed as it is the contract that can be reviewed by consumers. OpenAPI specification is one of the example in REST API.

Service providers need to think about how to make their services easily to be consumed. For example, each service will register itself on a centralized registry so that consumers can discover them by serviceId only. The consume sometimes can also pass in a tag to look for a special instance of the
service. For example, there are several instances of the same service that connect to different dbs for testing. Here is an [design article][] on this topic.  

Marketplace is another tool that can help consumers to find available services and contact service provider to speed up the process to consume it. 

#### Isolate Failure

Distributed system is much easier to make things fail. There are more network boundaries and more services increase the surface of failure.  

Also due to more services involve in a request, one bad service will lock up others and causing cascade failure which brought down the entire system. There are several techniques to lower the risks. For example, all services should be designed fail-fast with very short timeout. In Java EE
platform, circuit breaker, bulkhead need to be considered almost all the case. Another way is to change your architecture from request/response to event driven.   

#### Highly Observable

Active monitoring is OK if there are small number of services. Once your system scale to hundreds of services, you need reactive monitoring. The platform needs to provide centralized logging and metrics and there should be some stream processing on the incoming data to alert operation team something abnormal is happening. Also, the logs must be aggregated by a traceabilityId or correlationId so that the same transaction can be visible in logging aggregator tool such as ELK.

####  High Throughput, Low Latency and Small Memeory Footprint

The reason to adopt microservices architecture is to scale and it requires each service to be performed better. There are more services involved in a request so all services latencies will be added up to impact overall SLA. Unlike monolith, there might be thousands microservices instances running within your organization and smaller memory footprint will save a lot for production provision cost.
   
#### High Productivity

One of the reasons for microservices is to speed up the development and at the same time make it safe to change at scale.  

In a microservice architecture, the services tend to get simpler, but the architecture tends to get more complex. That complexity is often managed with tooling, automation, and process.

When you first begin learning about microservice architecture it’s easy to get caught up in the tangible parts of the solution. You don’t have to look hard to find people who are excited about Docker, continuous delivery, or service discovery. All of these things can help you to build a system that sounds like the microservice systems we’ve been discussing. But microservices can’t be achieved by focusing on a particular set of patterns, process, or tools. Instead, you’ll need to stay focused on the goal itself—a system that can make change easier.

More specifically, the real value of microservices is realized when we focus on two key aspects—speed and safety. Every single decision you make about your software development ends up as a trade-off that impacts these two ideals. Finding an effective balance between them at scale is what we call the microservices way.

* The Speed of Change
* The Safety of Change
* At Scale
* In Harmony

The original intent of the microservice architecture concept—to replace complex monolithic applications with software systems made of replaceable components. 


##### Speed
* Agility allows organizations to deliver new products, functions, and features more quickly and pivot more easily if needed.
* Composability reduces development time and provides a compound benefit through reusability over time.
* Comprehensibility of the software system simplifies development planning, increases accuracy, and allows new resources to come up to speed more quickly.
* Independent deployability of components gets new features into production more quickly and provides more flexible options for piloting and prototyping.
* Organizational alignment of services to teams reduces ramp-up time and encour‐ ages teams to build more complex products and features iteratively.
* Polyglotism permits the use of the right tools for the right task, thus accelerating technology introduction and increasing solution options.

##### Safety

* Greater e ciency in the software system reduces infrastructure costs and reduces the risk of capacity-related service outages.
* Independent manageability contributes to improved efficiency, and also reduces the need for scheduled downtime.
* Replaceability of components reduces the technical debt that can lead to aging, unreliable environments.
* Stronger resilience and higher availability ensure a good customer experience.
* Better runtime scalability allows the software system to grow or shrink with the business.
* Improved testability allows the business to mitigate implementation risks.


[consumer-driven contract]: /design/consume-contract/
[design article]: /design/env-segregation/
