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
---


### Non-functional requirements

##### Scalability


##### Security

When users log in, a JWT access token is generated from light-oauth2 for them, which is valid for 15 minutes. Along with the JWT access token, a refresh token will be issued and it will be valid within 24 hours to get a new JWT access token if necessary. By providing the access token for subsequent requests they can perform operation which require authentication.


##### Availability


##### Asynchronous


### Technology Stack

For backend services, all of them are built on top of light-rest-4j or light-hybrid-4j with light-eventuate-4j to adopt Event Sourcing and CQRS. 

For frontend, we are using React, Redux, Material UI and [react-schema-form][]


[react-schema-form]: https://github.com/networknt/react-schema-form

