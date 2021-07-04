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
reviewed: true
---

While working with our customers, we extracted the following patterns from user cases we encountered. The list is not exhaustive as this is a live document.

### Legacy Web App Consuming APIs

More and more organizations are in the process of transforming their monolithic applications to microservices. Given the size of the monolith, it is not possible to make the transition overnight. A typical approach is to move some components out of the monolith and implement them with microservices. The existing application will call the newly implemented services when needed. 

When adopting this approach, we want to ensure that the changes to the existing monolith are as minimum as possible. Any change can introduce defects in the monolith and the cost to stabilize it might be very high. 

Most traditional multi-tier web applications are using sessions for authentication and authorization; however, when consuming microservices, they need to provide JWT in the header to authenticate themselves, and for some services, additional user info is required in JWT for fine-grained authorization. What is the best approach to make a minimum change in the monolith and convert the user session to JWT to consume microservices? Switching from session to OAuth 2.0 and JWT is not an option as the changes are too significant for the entire monolithic. So the next logical choice is to convert the user session to JWT token as the session instance has all the information about the user. To do that, the OAuth 2.0 provider needs to support this with a customized grant type.

In [light-oauth2][] we have added a new client type called trusted along with public and confidential client types defined in the OAuth 2.0 specification. This client type supports a new grant type called [client_authenticated_user][]. There is also a [tutorial] to show you how to use this grant type. 

The existing legacy application still uses session which contains user info after successful authentication. Before calling the APIs, it will call the light-oauth2 token service to exchange a JWT token and refresh token with the user info. The JWT token will be expired, and it can always renew a new JWT token with the refresh token from light-oauth2 token service. In this design, we are making a minimum change to the existing application to meet the security requirement in consuming the APIs. 

From the end-user perspective, all the changes are transparent. The login session controls access to the application, and when the session expires, the JWT and refresh token in the session object are gone as well. Some customers also have a special logic to extend the session if there are activities from the end-user.

### Single Page App Consuming APIs

These days, more and more web applications are built with Javascript frameworks like React, Angular or Vue. Naturally, these browser applications will consume backend services which implement business logic. To protect the services, some authentication and authorization mechanisms must be applied. 

Most online tutorials are written by the front-end developers and OAuth 2.0 implicit grant type is used as the browser cannot keep the client_secret. This design is OK for online forums or blogs, but when building financial applications, implicit grant type cannot be trusted. As we are aiming for enterprise customers, a safer approach must be adopted in the design.

In all OAuth 2.0 grant types, the authorization code grant type is the safest; however, it requires that you have a server that is responsible for handling the authorization code and getting access tokens. For most financial organizations, exposing internal services to the SPAs directly is not allowed as the access is coming from the Internet. This requires a static IP address BFF (backend for frontend) service to serve the single page application and static content, manage security and perform service discovery to access internal services. In other words, it is acting as an API gateway but fully distributed as every single page application has its own API gateway.

When using the Light, there are three types of BFFs. 

* Existing legacy web server
* An aggregator that is built on the light-4j framework
* light-router which is a part of Light

The first type has been discussed in the above section for the legacy web server. The rest of two types are built on top of the Light which can leverage light-oauth2 authorization code flow and light-spa-4j [Stateless Auth][] handler. The stateless-auth middleware handler can be used in any API built with the light-4j frameworks. It handles the authentication with SPNEGO/Kerberos and downgrades to basic authentication. It also provides XSS and CSRF protections with OAuth 2.0 access token auto-renewal with the refresh token. 

Please refer to [SPNEGO/Kerberos authentication][] in light-oauth2 code service and [Stateless Auth][] middleware handler for details. 


[light-oauth2]: /service/oauth/
[client_authenticated_user]: /service/oauth/service/custom/
[tutorial]: /tutorial/oauth/custom/
[Stateless Auth]: /style/light-spa-4j/stateless-auth/
[SPNEGO/Kerberos authentication]: /service/oauth/service/spnego/

