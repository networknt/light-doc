---
title: "Token Handler"
date: 2018-05-29T18:40:13-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is a middleware handler that is responsible for getting a JWT access token from OAuth 2.0 provider for the particular router client. The only grant type that is supported in this handler is client credentials as there is no user profile information available here. It is designed for API to API invocation and the client side API cannot use client.jar from the light-4j platform. 

If any user profile is needed, the original client must follow the authorization code grant flow to get a token that represents the original user. If the original client is a single page application or native mobile application, the [StatelessAuthHandler][] can be used to follow authorization code grant type to get a token which contains both user_id, client_id, and scopes.

For any instance of light-router, either StatelessAuthHandler or TokenHandler is used in the middleware handler chain. If the light-router instance is not deployed on the client host or the instance is shared by other clients, TokenHandler cannot be used as the client_id/client_secret cannot be secured or there are multiple client_id/client_secret pairs. In this scenario, the original client must be responsible for getting the OAuth 2.0 token before calling the light-router. 

The client_id will be retrieved from client.yml and client_secret will be retrieved from secret.yml in order to get the JWT token. All endpoints to the OAuth 2.0 provider is defined in the client.yml as well. 

This handler will also be responsible for checking if the cached token is about to expire or not. If this is the case, it will renew the token in another thread, and the original request will be routed immediately. When a request comes, and the cached token is already expired, then it will block the request and all other requests and go to the OAuth 2.0 provider to get a new token and then resume all requests. 

The logic is very similar to client module in light-4j but this is implemented in a handler instead of a Java module.

This light-router is designed for standalones or clients that are not implemented in Java. Otherwise, you should use client module instead of this one. It should be used only if your client is not implemented in Java or the Java version is lower then JDK 8. 

There is no specific configuration file for this handler just to enable or disable it. If you want to bypass this handler, you can comment it out from service.yml middleware handler section or change the token.yml to disable it.

Once the token is retrieved from OAuth 2.0 provider, it will be placed in the header as Authorization Bearer token according to the OAuth 2.0 specification.

[StatelessAuthHandler]: /style/light-spa-4j/stateless-auth/
