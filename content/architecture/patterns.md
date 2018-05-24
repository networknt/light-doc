---
title: "Architecture Patterns"
date: 2018-05-24T05:19:11-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
---

While working with our customers, we have extracted the following patterns from use cases we encountered. The list is not exhausted as this is a live document. 

### Legacy Web App Consuming APIs

More and more organizations are in a process to transform their monolithic applications to microservices. Given the size of the monolith, it is not possible to make the transition overnight. A typical approach is to move some components our of the monolith and implement them with microservices. The existing application will call the newly implemented services when needed. 

When adopting this approach, we want to ensure that the changes to the existing monolith are as minimum as possible. Any change can introduce defects in the monolith and the cost to stabilize it might be very high. 

Most traditional multi-tier web applications are using sessions for authentication and authorization; however, when consuming microservices, it needs to provide JWT in the header to authenticate itself, and for some services, it requires additional user info in JWT for fine-grained authorization. What is the best approach to make a minimum change in the monolith and convert the user session to JWT to consume microservices? Switch from session to OAuth 2.0 and JWT is not an option as the changes are too significant for entire monolithic. So the next logical choice is to convert the user session to JWT token as the session instance has all the information about the user.  To do that, the OAuth 2.0 provider needs to support this with a customized grant type.

In [light-oauth2][] we have added a new client type called trusted along with public and confidential client types defined in the OAuth 2.0 specification. This client type supports a new grant type called [client_authenticated_user][]. There is also a [tutorial] to show you how to use this grant type. 

The existing legacy application still uses session which contains user info after successful authentication. Before calling the APIs, it will call the light-oauth2 token service to exchange a JWT token and refresh token with the user info. The JWT token will be expired, and it can always renew a new JWT token with the refresh token from light-oauth2 token service. In this design, we are making a minimum change to the existing application to meet the security requirement in consuming the APIs. 

From the end-user perspective, all the changes are transparent. The login session controls access to the application, and when session expired, the JWT and refresh token in the session object are gone as well. Some customers also have a special logic to extend the session if there are activities form the end-user. 

### Single Page App Consuming APIs



[light-oauth2]: /service/oauth/
[client_authenticated_user]: /service/oauth/service/custom/
[tutorial]: /tutorial/oauth/custom/
