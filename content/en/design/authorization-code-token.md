---
title: "Authorization Code Token"
date: 2023-07-14T12:25:43-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Most APIs built with light-4j or protected by http-sidecar or light-gateway deal with two types of OAuth 2.0 tokens: Client Credentials or Authorization Code. 

To verify the client credentials token, the JwtVerifyHandler should be enough to verify the token signature, expiration and endpoint scope against the specification. 

However, when we deal with the authorization code token, we might need to do a little bit more than the normal JWT token verification as the authorization code token contains the user-related claims, for example, userId, roles, AD groups etc.

To enforce the authorization at the role level or user level, we need to leverage the [fine-grained authorization][] AccessControlHandler in light-rest-4j.

[Here](/tutorial/security/fine-grained/) is a tutorial that shows how to use the AccessControlHandler and Rule Engine to authorize user access based on the Action Directory groups in the JWT token claims. 


[fine-grained authorization]: /design/fine-grained-authorization/
