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
---

This portal service can be found at [https://github.com/networknt/light-portal](https://github.com/networknt/light-portal)

This service is part of light-portal, and it is used for managing users for the light-4j platform. And users can get the service to run separately for user management for their own project.

### Scope

User Management Service only manages user create, confirm, update, delete and log in. The user role and authorization will be implemented in [light-oauth2][]. And the session management will be implemented in [light-session-4j][] framework.


### Non-functional requirements

User stories usually don't define non-functional requirements, such as security, development principles, technology stack, etc. So let's list them here separately.

The domain model is implemented in pure Java using Domain-driven design principles and its independent of the underlying technology stack.

When users log in, a JWT token is generated for them, which is valid for 15 minutes. Also, a refresh token will be issued with 24 hours expiration. By providing the JWT access token for subsequent requests they can perform operations which require authentication for 15 minutes. Before the access token is about to be expired, a new access token will be retrieved by sending the refresh token to the token service. 

Password reset tokens are valid for 10 minutes and email address confirmation tokens for a day.

Passwords are encrypted with a cryptographically strong algorithm (Bcrypt) with a per-user salt.

GDPR(General Data Protection Regulation) compliance. That means we cannot use event sourcing with Kafka only but need a database so that user info can be deleted if necessary. For the current implementation, we are using light-eventuate-4j, so the aggregation is in a MySQL database. Of course, users can have their implementation if they want some extra features. 


### Basic requirement & use case:

As a user, I want to register, so that I can get access to content which requires registration

As a user, I want to confirm my email address after registration

As a user, I want to login and log out

As a user, I want to change my password

As a user, I want to change my email address

As a user, I want to be able to reset my password, so that I can login after I lost my password

As a user, I want to update my profile, so that I can provide my correct contact data

As a user, I want to close my account, so that I can close my relationship with that service I signed up for

As an admin, I want to manage (create/delete/update) users manually, so that staff members wouldn't have to go over the registration process

As an admin, I want to create users manually, so that staff members wouldn't have to go over the registration process

As an admin, I want to list all users, even those once who closed their account

As an admin, I want to be able to see users' activity (login, logout, password reset, confirmation, profile update), so that I can comply with external audit requirements


### Workflow


![User_registration_workflow](/images/User_registration_workflow.png)


### Service Rest Api

GET /user/{user_id}

Finds a user with the given ID.

GET /user  

List all users in the system

POST /user

Registers a new user

DELETE /user/{user_id}

Deletes the given user.

PUT /user/{user_id}

Updates the profile of a given user.

GET /user/tokens/{user_id}?token={token_id}

Uses the token of the given user and performs the action related to the token�s type.

GET /user/email?email={email}

Finds a user with the Email Address.
  
GET /user/name?name={name}

Finds a user with the login name.

PUT /user/login

Login user with given login form


### Project modules

### Common module:

common module includes all common objects for the service:

Entities  -- Entities have got a clear identify and lifecycle which needs to be managed

User,
ConfirmationToken

Value Objects:

In contrast to entities, value objects don't have a clear identity, that is, they are just grouping a set of attributes and if these attributes are the same as the attributes of another value object of the same type, then we can treat them the same

 AddressData, 
 AuditData, 
 ContactData,
 Password,
 UserDto

Event  -- handle event and process it by light-eventuate-4j framework

UserSignUpEvent,
UserSignUpSuccessEvent,
UserSignUpFailEvent,
UserUpdateEvent,
UserDeleteEvent,
UserActionEvent

Util    -- help and util classes

IdGenerator (generate unique Id),
LocalDateTimeUtil (time util class).
Validator (validate user input email, password etc),
EmailSender  (send confirm email for new signup user or email change)


### auth module:

auth module includes the service class for user authentication, and command side component for eventuate system.

command  -- Aggregates and command class for light-eventuate-4j framework to handle events.

service  -- java service POJO class for user signup and user authentication





### We provide two types of services:

### Normal microservice:

It can be run local to and persist the user info to local database only.

Module:

usermanagement-service


### Event sourcing microservice:

Integrate the service with light-eventuate-4j framework to process user management with event sourcing. It include command side service and query side service.

Module:

rest-coomand    -- command side service

rest-query      -- query side service



[light-oauth2]: /service/oauth/
[light-session-4j]: /style/light-session-4j/
