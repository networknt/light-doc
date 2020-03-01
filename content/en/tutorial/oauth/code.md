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
reviewed: true
---


This is part of the authorization flow that takes the user's credentials and redirect back authorization code to the webserver through a user agent (browser or mobile phone). The web server will use the authorization code along with client_id and client_secret to get the access token and refresh token. 

In this tutorial, we are using the curl command to access the service for demo purposes. In reality, this should be done by a [login view][] single page application. The light portal https://signin.lightapi.net is an example of a login view. 

There are two endpoints: /oauth2/code@get and /oauth2/code@post

The GET endpoint uses Basic Authorization, and POST endpoint uses [Form Authorization](/tutorial/oauth/form-auth-local/).

When the GET endpoint is used, it provides a popup window on the browser to ask username and password. And there is no need to create a login page and error page. This is the most simple approach but with a lot of limitations. For production usage, a form-based login application with a POST endpoint is recommended. 

The POST endpoint is usually used when you want to have a customized login application to make sure users have the same experience as they browse other parts of your web server. The browser will have a login form to collect user credentials and posts it to the OAuth2 server /oauth2/code endpoint. Once the user is authenticated, an authorization code is redirected back to the browser with a redirect URI passed in from the request, or the default redirect URI for the client will be used from client registration. As you might guess, this endpoint requires customization most of the time on the login page and error page. The default [login view][] application is provided as a starting point to make your customized login application.


There is only one bootstrap admin user after the system is installed, and the default password is "123456". The password needs to be reset immediately with User Service for production.  

To get authorization code put the following URL into your browser.

```
https://localhost:6881/oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization
```

If this is the first time you hit this URL on the browser, you will have a popup window for user credentials. Now let's use admin/123456 to login given you haven't reset the password yet for the admin user. If you have logged in recently, your credentials are cached and no popup window will show up. 

Once authentication is completed, an authorization code will be redirected to your browser's address bar—something like the following. Don't worry about the error `This site can't be reached` because you don't have a server to serve the http://localhost:8080 locally.  What we are doing here is just demonstrate the flow of the Authorization code flow, and the real application will be started from a browser application. 

```
http://localhost:8080/authorization?code=pVk10fdsTiiJ1HdUlV4y1g
```

If you want to call the get endpoint from your command line or script, you can put the user credentials into the header in the command. The light-oauth2 code service GET endpoint accepts Basic authentication in the Authorization header in base64 format, so we need to encode the admin:123456 into base64. There are some online encoders available and here is one: https://www.base64encode.org/

After encoding the credentials, we got `YWRtaW46MTIzNDU2`. Here is a sample curl command with -v to output the redirect 302 response. If there is no -v, then the result is empty on the terminal.

```
curl -k -v -H 'Authorization: BASIC YWRtaW46MTIzNDU2' 'https://localhost:6881/oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization'
``` 

And here is an example of a result. 

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 6881 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  start date: Jan 19 14:27:52 2013 GMT
*  expire date: Jan 17 14:27:52 2023 GMT
*  issuer: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x564edc4ed580)
> GET /oauth2/code?response_type=code&client_id=f7d42348-c647-4efb-a52d-4c5787421e72&redirect_uri=http://localhost:8080/authorization HTTP/2
> Host: localhost:6881
> User-Agent: curl/7.58.0
> Accept: */*
> Authorization: BASIC YWRtaW46MTIzNDU2
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 302 
< server: L
< location: http://localhost:8080/authorization?code=xTpGcT3USO2_BNBd8xVCFQ
< content-length: 0
< date: Sun, 01 Mar 2020 16:39:35 GMT
< 
* Connection #0 to host localhost left intact
```

As you can see, the response code is 302 redirect, and the redirect location is http://localhost:8080/authorization?code=xTpGcT3USO2_BNBd8xVCFQ

In this tutorial, we are just show you how to verify that the code service is working. For a real example, please take a look at the open banking [client][] tutorial.


[login view]: /tutorial/oauth/login-view/
[client]: /tutorial/open-banking/client/

