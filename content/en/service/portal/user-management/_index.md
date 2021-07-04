---
title: "User Management"
date: 2018-03-03T20:18:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This service is part of light-portal, and it is used for managing users for the light platform. Most functionalities of the light-portal need the user to be logged in. 


### Scope

User Management only manages user registrations, email confirmations, profile updates, user deletes and password changes. The user login and logout will be managed by the light-spa-4j and light-oauth2 on light-portal for single-page applications. When you scale to multiple router instances, the session management will be implemented with [light-session-4j][].

### Workflow

![User_registration_workflow](/images/User_registration_workflow.png)

### Modules

All light-portal services are based on the light-kafka with Event Sourcing and CQRS. There are two modules that is involved for user management. 

##### user-command

It is responsible for updates with the user profile and based on light-hybrid-4j for the API with light-kafka for the event store.

##### user-query

It is responsible for query user profiles with different material view projections generated from the user events. All the data is readily available in the Kafka Streams Key/Value store, so the latency is very low. 

* [User Registration](/service/portal/user-management/registration/)
* [Update Profile](/service/portal/user-management/profile/)



[light-oauth2]: /service/oauth/
[light-session-4j]: /style/light-session-4j/
