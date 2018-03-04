---
title: "User Management Introduction"
date: 2018-03-03T20:18:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

# Introduction

This portal service can be found at [https://github.com/networknt/light-portal](https://github.com/networknt/light-portal)

This service is part of light-portal and it is used for managing user for light-4j. And user can get the service to run separate for user management for their own project.

## Scoop

User Management Service only manage user create, confirm, update, delete and login. The user role and authorization will be implemented in light-oauth.

And the session management will be implement in light-4j framework


## Non-functional requirements

User stories usually don't define non-functional requirements, such as security, development principles, technology stack, etc.  So let's list them here separately.

The domain model is implemented in pure Java using Domain-driven design principles and its independent of the underlying technology stack to be used

When users log in, a JWT token is generated for them, which is valid for 24 hours. By providing this token for subsequent requests they can perform operation which require authentication

Password reset tokens are valid for 10 minutes and email address confirmation tokens for a day

Passwords are encrypted with a cryptographically strong algorithm (Bcrypt) with per-user salt

A RESTful API is provided for interacting with the user registration service



## Basic requirement & use case:

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


## Workflow


![User_registration_workflow](/images/User_registration_workflow.png)


## Service Rest Api

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


## Project modules

## Common module:

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


## auth module:

auth module includes the service class for user authentication, and command side component for eventuate system.

command  -- Aggregates and command class for light-eventuate-4j framework to handle events.

service  -- java service POJO class for user signup and user authentication





## We provide two types of services:

## Normal microservice:

It can be run local to and persist the user info to local database only.

Module:

usermanagement-service


## Event sourcing microservice:

Integrate the service with light-eventuate-4j framework to process user management with event sourcing. It include command side service and query side service.

Module:

rest-coomand    -- command side service

rest-query      -- query side service




## End

