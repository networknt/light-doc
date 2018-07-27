---
title: "Authenticator"
date: 2018-07-27T11:12:10-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For every organization, there would be a lot of different authentication/authorization mechanisms in use. For example, Kerberos/SPNEGO SSO will be used for employee and a customer database will be used for customers. In a big organization like a bank, almost every client application will have its own auth mechanism. This requires that OAuth 2.0 providers support customization and to plug in different authentication/authorization providers. Most of the commercial products don't have source code available, and it requires to manipulate security policies to do any customization. 

The light-oauth2 has a module call authhub which contains all the built-in auth providers. It also provides an abstraction layer for users to customize existing auth providers or create their own auth providers. For each individual deployment, the end user can choose which auth providers to be loaded in the externalized service.yml config file.

When registering an individual client, there is an option to choose which auth provider to be used for the client. If no auth provider is chosen, then the default auth provider will be used for the code service. Currently, the default implementation is Kerberos/SPNEGO SSO with basic authentication of user service fallback because most organizations are using Microsoft Active Directory to manage employee authentication and a customer table for customers. 

Every auth provider contains two different parts or two different phases. 

### Authentication

This is the action that verifies the user credentials. Different auth provide can decide how to implement it. It can be basic authentication based on a user table or active directory. It can be Kerberos/SPNEGO SSO for all web applications running on the same browser. It can be several mechanisms combined together to handle more complicated scenarios like internal users authenticated by Kerberos/SPNEGO with basic backup and external users authenticated by a customized database or service. 

### Authorization

Once the authentication phase is done, the next step is the authorization. When resource server receives JWT token, it must perform extra fine-grained authorization to determine if the access is granted within the business context. This requires that the JWT token needs to contain extra information about the user or client. 

For the client, light-oauth2 support custom claims in JSON format during client registration and it will be part of the JWT token generated for the client. 

For the user, the auth provider will try to get extra user info after the authentication is done. For example, if LDAP is used, the AD group will be retrieved with a function id user to put into the role section in the JWT token. 

### Auth Providers

- [default][]
- [marketplace][]

[default]: /service/oauth/authenticator/default-auth/
[marketplace]: /service/oauth/authenticator/marketplace-auth/




