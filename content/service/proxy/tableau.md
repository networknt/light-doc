---
title: "Tableau Authentication Handler"
date: 2018-05-02T22:58:58-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-proxy is used to bring legacy APIs to the light ecosystem by providing cross-cutting concerns on the network level. One of the cross-cutting concerns is the security with JWT verification. It helps to protect the downstream API which doesn't support OAuth 2.0 with JWT access token; however, most APIs do have some security mechanisms to authenticate and authorize access. Some of them even have customized security with its login API and customized token like Tableau. 

While working with one of our customers to integrate with downstream API exposed from Tableau server, we found that it is not comfortable and secure to authenticate the access from the B2B client. It would be much easier to build a security layer in the light-proxy so that we can shield the access from outside only to specific endpoints instead give them the site admin user privilege. 

To meet the above requirement, we have built a customized authentication handler which can handle the login and pass the token to the downstream Tableau server.  

The logic is straightforward in the handler. It requires the B2B client to pass the Tableau Content URL in the header named tableauContentUrl. Based on the passed in value in the header, it picks up the Tableau user from tableau.yml and credential from secret.yml to log in to the Tableau server. Then the token is retrieved from the response of the login request and put into the header for the current client request. When the request reaches the Tableau server, it has the authentication token in the header already. 

The implementation is simple and enough for the use case (batch process) we have; however, it is not the most efficient way to handler the security as every request needs to go to the Tableau to log in. 

If this is a real-time API proxy, we need to have a map to cache the token for each Tableau Content URL and reuse the cached token until it is expired. The logic is a little bit complex than the client module in light-4j as there are many tokens and expirations to be managed but it is doable. 

Here is the configuration file for this handler. 

tableau.yml 

```
# Enable tableau handler or not, default to false
enabled: false
# The tableau server url.
serverUrl: http://127.0.0.1:8080
# The tableau signin API path
serverPath: /api/2.8/auth/signin
# Tableau Credentials for signin
# Tableau user name
tableauUsername: "admin"

```
