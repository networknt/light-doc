---
title: "Light Portal"
date: 2018-01-14T21:14:46-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 20
weight: 5	#rem
aliases: []
toc: false
draft: false
---

A service for consumer and provider runtime management and marketplace to bridge between consumers 
and providers. When you build microservices in large scale on top of light-4j and related frameworks,
you need a service to manage the entire life-cycle of your services and provide a market place for
bridging API consumers and API providers. We have explore the open sources projects on github.com
and could not find any existing product that meets our requirement, so we decide to build it on top
of light-4j framework.   

The following are major components for light-portal:

### API Certification

This service is part of light-portal and it is used to check if the production deployment meet the security requirements.

The following things will be checked with server/info endpoint:

If TLS https is enable and key pair are replaced
If http port is disabled
If JWT token verification and scope verification are enabled
If metrics is enabled
If the right version of JDK/JRE is used to start the server
If the default JWT key is replaced



### User Management

This service is part of light-portal and it is used for managing user for light-4j. And user can get the service to run separate for user management for their own project.

We provide two types of services:

##### Normal microservice:

It can be run local to and persist the user info to local database only

##### Event sourcing microservice:

Integrate the service with light-eventuate-4j framework to process user management with event sourcing. It include command side service and query side service.


## Non-functional requirements

User stories usually don't define non-functional requirements, such as security, development principles, technology stack, etc.  So let's list them here separately.

The domain model is implemented in pure Java using Domain-driven design principles and its independent of the underlying technology stack to be used

When users log in, a JWT token is generated for them, which is valid for 24 hours. By providing this token for subsequent requests they can perform operation which require authentication

Password reset tokens are valid for 10 minutes and email address confirmation tokens for a day

Passwords are encrypted with a cryptographically strong algorithm (Bcrypt) with per-user salt

A RESTful API is provided for interacting with the user registration service

