---
title: "Authorization Code"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 30
aliases: []
toc: false
draft: false
---


This is part of authorization flow that takes user's credentials and redirect back authorization
code to the webserver through user agent (browser). The web server will use the authorization code
along with client_id and client_secret. 

In this tutorial we are using curl command to access the service for demo purpose. In reality,
this should be done in the light-portal login page. 


There are two endpoints: /oauth2/code@get and /oauth2/code@post

The GET endpoint uses Basic Authorization and POST endpoint uses Form Authorization.

In most of the cases, you should use GET endpoint as it provides popup window on
the browser to ask username and password. And there is no need to create a login page
and error page.

POST endpoint is usually used when you want to have customized login page and error page to make 
sure users have the same experience as they browser other part of your web server. Browser will have 
a login form to collect user credentials and posts it to the OAuth2 server endpoint. Once the user
is authenticated, a authorization code is redirected back to the browser with a redirect URI passed
in from the request or the default redirect URI for the client will be used from client registration.
As you might guess, this endpoint requires customization most of the time on login page and error page.
Default login page and error page are provided as a starting points to make your customized pages.


There is only one admin user after the system is installed and the default password
is "123456". The password needs to be reset immediately with User Service for
production.  

To get authorization code put the following url into your browser.

```
https://localhost:6881/oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization
```

If this is the first time you hit this url on the browser, you will have a popup window for user
credentials. Now let's use admin/123456 to login given you haven't reset the password
yet for admin user.

Once authentication is completed, an authorization code will be redirect to your
browser. Something like the following.

```
http://localhost:8080/authorization?code=pVk10fdsTiiJ1HdUlV4y1g
```

If you want to call the get endpoint from your command line or script, you can put
the user credentials into the header in above command. Just make sure you have 
a server listening to the redirect uri you have specified. 

Here is a sample curl command.

```
curl -k -H "Authorization: Basic admin:123456" https://localhost:6881/oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization
``` 
If you want to try the above command line, you have to make sure that redirect_uri is alive. Otherwise,
you will have an error that doesn't make any sense.

