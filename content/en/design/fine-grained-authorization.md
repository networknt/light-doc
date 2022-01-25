---
title: "Fine Grained Authorization"
date: 2021-11-11T14:42:27-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


### Layered Security

When we are working on API security design, multiple layers need to be carefully considered. Just like an onion, you can peel multiple layers until you reach the core. 

### Technical Concerns

First, the JWT verifier handler will ensure that the authorization header has a valid, unexpired token with the correct signature. It will allow the consumer to invoke the API at the API level. 

Second, the JWT verifier also validates the scope in the JWT token against the scopes defined in the OpenAPI specification to ensure the JWT token has access to the current endpoint. This is the endpoint level security. 

The above two layers of security happen at the technical level, and it is common to any API and organization. A standard implementation has been done in the security module in the light-4j open-source repository.


### Business Concerns

After the JWT verifier handler verifies the request, the security check goes into the business domain, and this will be different from API to API and organization to organization. 

The following design is based on the generic requirements for financial organizations like banks. You can follow the same pattern or customize it for other organizations if the security requirement is different. 


### Rule-based Authorization

The fine-grained authorization for APIs is very complicated, and traditional role-based and attribute-based authorization cannot fulfill the requirement. You need to run a list of business rules to calculate if a user has permission for an API request. Let's take a look at the rule-based authorization requirement, design and implementation. 

##### Requirement

Let's list several common requirements about rule-based authorization at the API endpoint level. 

Request Side

* Allow Client Credentails Token access even without user_id claim. However, a sid (serviceId) claim is not null and it equals to one of the values in the request (a path parameter or a property in the body). This is for API to API invocation. For example, during the API startup, it register to the controller and access config server to get its configuration. 

* Allow certain roles to access. For example, only manager role can create a new account. 

* Allow certain roles to access their own data. For example, a user role can update only his/her user profile. 

Response Side

* A manager role can see extra data elements than a user role. For example, a manager returns 27 columns and a teller can only return 16 columns for customer account query. 
* A manager role can see more rows than a user role. For example, a manager can see all accounts for a customer and a teller can only see customers personal accounts. 

##### Design

The most important thing we have to resolve is how the service get the rule-based authroization definition at runtime. Like the scopes definition for endpoints, the best location would be the OpenAPI specification as light-4j is using it at the runtime. However, the current specification and tool chains don't support it at the moment and we have to use the extension of OpenAPI. We are lucky to have our own implementation of OpenAPI parser to parse the extension at runtime and use the cached information to perform role-based authorization.

```
paths:
  /accounts:
    get:
      summary: "List all accounts"
      operationId: "listAccounts"
      x-request-access:
        rule: "account-cc-group-role-auth"
        roles: "manager teller customer"
      x-response-filter:
        rule: "account-row-filter"
        teller: 
          status: open
        customer:
          status: open
          owner: @user_id          
        rule: "account-col-filter"
          teller: ["num","owner","type","firstName","lastName","status"]
          customer: ["num","owner","type","firstName","lastName"]
      security:
      - account_auth:
        - "account.r"

```

We have a rule engine called yaml-rule and the rule we are using is defined as the following for the x-request-access

```
account-cc-group-role-auth:
  ruleId: account-cc-group-role-auth
  host: lightapi.net
  description: Role-based authorization rule for account service and allow cc token and transform group to role.
  conditions:
    - conditionId: allow-cc
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.user_id
      operatorCode: NIL
      joinCode: OR
      index: 1
    - conditionId: manager
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.groups
      operatorCode: CS
      joinCode: OR
      index: 2
      conditionValues:
        - conditionValueId: manager
          conditionValue: admin
    - conditionId: teller
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.groups
      operatorCode: CS
      joinCode: OR
      index: 3
      conditionValues:
        - conditionValueId: teller
          conditionValue: frontOffice
    - conditionId: allow-role-jwt
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.roles
      operatorCode: NNIL
      joinCode: OR
      index: 4
  actions:
    - actionId: match-role
      actionClassName: com.networknt.rule.FineGrainedAuthAction
      actionValues:
        - actionValueId: roles
          value: $roles

```

All rules are managed by the light-portal and shared by all the services. 

In light-4j, we have a startup hook in the rule-loader module to load the rules for the service during the startup. 

In the light-rest-4j we have a module called access-control that is responsible for the fine-grained access control. 
