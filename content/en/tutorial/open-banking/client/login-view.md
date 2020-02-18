---
title: "Light-OAuth2 Login View"
date: 2020-02-14T23:59:39-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have made [local-consul configuration][] changes for the light-oauth2 and restarted all the microservices. In this step, we are going to walk through the [login-view][] of the light-oauth2, which accepts the username and password credentials and returns the authorization code in the OAuth 2.0 grant flow. 

The login-view is based on the redirect flow for Single Page Applications. 

* The original application will have a log in button or menu item. Once it is clicked, the browser will redirect to the login-view application, which is based on the React.

* The login-view will accept username and password to invoke light-oauth2 code service. 

* The code service will redirect to the /authorization?code=xxx to the browser. 

* The browser will invoke the above API which is handled by StatelessAuthHandler in light-spa-4j

* The StatelessHandler will use the code query parameter, client_id, client_secret saved on router server to get access token and refresh token from the light-oauth2 token service. The StatelessHandler will save the access token, refresh token, csrf, userId, roles, and userType as cookies. The access token and refresh token are HttpOnly and Secure, so the browser application cannot access them. The handler also returns the redirectUri, denyUri, and scopes. 

* The login-view will use the response to display a consent page will a list of scopes and ask users to DENY or ACCEPT the grant. 

* If DENY button is clicked, all the cookies will be removed, and the browser will be redirected to the denyUri. The entire flow is canceled. 

* If ACCEPT button is clicked, the browser will be redirected to the redirectUri. 

* All subsequent calls to the light-router server from the original SPA will carry all the cookies to the request. The StatelessAuthHandler will capture the access token and refresh token and put the access token into the request header to invoke downstream APIs. 

* If the access token is expired or about to expire, The StatelessAuthHandler will renew a new token and put the new one into the request header. 

* To prevent CSRF(Cross-site request forgery), the original application must get the csrf cookies and put it into the request header for each request. 

Above is a simplified flow without a lot of detail for easy to understand. 

Here is a video walkthrough of the authorization code flow and the login-view single page application. 


{{< youtube VdWqymAKslg >}}


In the next step, we are going to walk through the [light-oauth2][] configuration and start all services locally. 

[local-consul configuration]: /tutorial/open-banking/client/oauth-config/
[login-view]: https://github.com/networknt/light-oauth2/tree/master/login-view
[light-oauth2]: /tutorial/open-banking/client/oauth-config/

