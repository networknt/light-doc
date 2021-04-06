---
title: "Why check token expiration"
date: 2018-02-13T20:50:25-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "design"
    weight: 70
weight: 70
aliases: []
toc: false
draft: false
reviewed: true
---

In web service architecture, normally people handle JWT token expiration re-actively. Here
is the flow. 

* Client sends a request with a JWT token in the header
* The service receives the request and verifies if the JWT token expired
* If expired, then returns 401 - token expired
* When the client receives this error and body, it will go to the OAuth 2.0 provider to renew a new token
* Resends the request with the new token

Note that if token is not expired then go to the next step. 

The above flow doesn’t work with microservices architecture as it will cause data consistency issues if the token being used is about to expire.

For example, the client is calling two services in sequence. The first service verifies the token, making sure it is not expired and that the transaction went through. However, when the second service receives the token, it is already expired and returns an error message to the client. In this scenario, the client can still renew the token and resend the second request with the new token.

Let’s take a look at the next scenario. The client is calling the first service and the first service calls the second service. What if the first service passed but the second service got a token expired error? If we still follow the above flow, then we need the client to renew the token and retry first service again then first service calls to second service. This requires the first service must be idempotent which is much more complicated to handle.

To avoid these complicated scenarios, in light-4j framework, we check the token expiration pro-actively in the [client][] module and renew a new token before it is about to expire. The default configuration is 1 minute before token expiration. A separate thread will contact [light-oauth2][] token service to renew the token.

[client]: /concern/client/
[light-oauth2]: /service/oauth/
