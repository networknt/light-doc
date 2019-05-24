---
title: "Technology Stack"
date: 2019-03-07T12:32:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


### Non-functional requirements

##### Scalability


##### Security

When users log in, a JWT access token is generated from light-oauth2 for them, which is valid for 15 minutes. Along with the JWT access token, a refresh token will be issued and it will be valid within 24 hours to get a new JWT access token if necessary. By providing the access token for subsequent requests they can perform operation which require authentication.


##### Availability


##### Asynchronous


##### Plugin

The entire portal is implemented in a plugin architecture, and all backend services are implemented as microservices based on well-designed API specifications. On the UI view, menu items are customized per user or per organization so that only a subset of services are available to most users or organizations. We provide an implementation for the specification of each service; however, any contributor can create their implementations as long as they respect the same contract. For example, our user-management service is built on top of the light-eventuate-4j framework with Event Sourcing and CQRS; however, a customer can easily implement the same specification with a database solution for user-management and replace the default implementation. 

There are many reasons that a company needs to implement the service differently. For example, the local regulation with some non-functional requirements. Some company has a policy that can only choose a product from an approval list, etc. 

### Technology Stack

For backend services, all of them are built on top of light-rest-4j or light-hybrid-4j with light-eventuate-4j to adopt Event Sourcing and CQRS. 

For frontend, we are using React, Redux, Material UI and [react-schema-form][]


[react-schema-form]: https://github.com/networknt/react-schema-form

