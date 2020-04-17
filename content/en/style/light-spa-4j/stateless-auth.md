---
title: "Stateless Auth Handler"
date: 2018-05-24T08:03:54-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is the handler that is responsible for handling redirected authorization code from light-oauth2 code service after SPNEGO/Kerberos or Basic authentication or Form authentication.  It is also responsible for verifying the subsequent requests, prevent XSS and CSRF attacks, renew JWT tokens with the refresh token when the previous JWT token is expired. 

It is called stateless auth handler because it doesn't need any stateful session on the BFF server so that the BFF can be scaled freely. The JWT access token and refresh token are sent to the browser with httpOnly and secure cookies. It ensures that the Single Page Application cannot access the cookies, and the cookies can only be sent when HTTPS is enabled. 

### Authorization

In the config file statelessAuth.yml, there is an authPath property to define which path is for the OAuth 2.0 authorization code handling. If the incoming request has the match request path with authPath in the config, it will trigger the authorization code handling. 

* It first checks if the query parameter code exists. If not, return an error to the browser.

* If code exists, create a UUID CSRF token and call light-oauth2 token service to get access token. The CSRF token will be part of the access token (JWT custom claim).

* Return error is there is an error in getting the access token. 

* If the access token and refresh token are retrieved from light-oauth2 token service, return both to the browser with httpOnly and secure cookies. Given the cookies are httpOnly, the SPA Javascript cannot access the cookies but these cookies will be passed to the server for each subsequent API request.  

* The generated CSRF token will be returned to the browser with a secure cookie. The SPA can access this CSRF token and will put this token in the subsequent request in header X-CSRF-TOKEN. 

* The response is a redirect and the URI is defined in the statelessAuth.yml as redirectUri. The SPA need to have a route for this path and switch to the post-login page or protected page etc. 

Although we are sending the tokens (both access and refresh tokens) to the browser, the Javascript cannot access them. So there is no risk for the cross-site scripting attack. 

The cookies are subject to cross-site request forgery attack so we have sent a CSRF token to the SPA and ask the SPA to include this token in the request header for subsequent requests. This token in the header will be compared with the one in JWT  token on the stateless-auth handler. 

### Verification

If the incoming request path is not authPath, then the JWT token and CSRF token verification process are started. 

* First, the access token is retrieved from the cookie. If there is no access token, the refresh token will be tried to renew the token. If there is no refresh token neither, the SPA will be redirected to the login page which is defined in the statelessAuth.yml as cookieTimeoutUri. When this happens most likely, the cookies are timed out. 

* If access token exists, the signature is verified and expiration time is verified as well. 

* The CSRF token will be retrieved from the JWT token and compare with the CSRF token retrieved from the X-CSRF-TOKEN request header. If any of the CSFR tokens is missing or both tokens are not matched, an error will be returned. 

### Token Renewal

If the access token is expired, then a token renew process is triggered. 

* If the access token is expired during the token verification, the refresh token will be retrieved from the refresh token cookie.

* Generate a new CSRF token and send the refresh token request to the light-oauth2 token service. The grant type is called refresh token in this case. We are not reusing the same CSRF token but regenerate a new one each time the access token is renewed. 

* Return error if token renewal fails. 

* If renewal is successful, send both the new access token and the new refresh token to the browser with httpOnly and secure cookies. The CSRF token will be sent with secure cookies to the browser. 


### Session Control

As you can see, we don't have a session object on the server side for each user, but we can maintain the user session with the access token and refresh token cookies. The expiration time of the access token is used to control how long the session should be kept. It is usually won't exceed 15 minutes. If you checked remember on the login page, then the refresh token will be kept in cookie for 90 days and it will be used to renew access token until it is expired. 

### Reference App

To help developers to learn how to use the light platform for developing Single Page App that consumes APIs, we have created a reference SPA application using React. The source code can be found at the following location. 

https://github.com/networknt/light-example-4j/tree/master/spa/react-stateless





