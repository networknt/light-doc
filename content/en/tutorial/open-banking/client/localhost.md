---
title: "Localhost Hot Reload for Developers"
date: 2020-02-25T11:32:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous steps, we are using virtual host ob.lightapi.net to test the react-client for the Open Banking services. For a simple React application development, it is OK; however, if there are too many components that need to be modified and tested. By building the react application into the build folder, copy to the light-rest-config folder and restart the light-router instance, it is not efficient. 

For the developer's productivity, we need to find a way to start the React application with the builtin Nodejs server and proxy the API calls to the light-router instance running locally. This allows us to make changes in the Javascript code, and the browser will be automatically reloaded. 

Here are the steps to get everything set up locally on a Ubuntu Linux 18.04 LTS.

### Environment

Before starting any services, we need to ensure that the environment is ready.

Update/etc/hosts to add the following line so that both ob.lightapi.net and obsignin.lightapi.net will be mapped to the local IP address. You need to change this to your local IP if you want to try the tutorial on your desktop. 


```
192.168.1.144   ob.lightapi.net obsignin.lightapi.net
```

Add an iptables rule to allow redirect from 443 to 8443 locally as the light-router instance will be started to listen to 8443 on HTTPS. However, we want to access it with normal HTTPS port 443. For more information, please refer to this [tutorial](/tutorial/security/port443/)

```
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -o lo -j REDIRECT --to-port 8443
```

### Consul

To start the local Consul server in docker.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

### Light-oauth2

To start the light-oauth2 instances locally and register them to the Consul.

```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose up -d
```

### Open Banking Services

To start the Open Banking services and register them to the local Consul server.

```
cd ~/open-banking/light-config-test/local-consul
docker-compose up -d
```

### Light-router

There are two different ways to start the light-router instance: standalone and docker. I would prefer to start it in standalone mode in an IDE so that I can debug it while working on the UI. If you are just a UI developer, start it with docker-compose will be the best. 

```
cd ~/networknt/light-config-test/light-router/local-openbanking
docker-compose up -d
```

To start the light-router instance in IDE with debug configuration files, please refer to the [debug tutorial](/tutorial/open-banking/client/debug-router/).

To support requests from https://localhost:3000, we need to make the following modifications in the configuration. 

statelessAuth.yml

```
# This handler is generic request handler for the OAuth 2.0 provider authorization code redirect.
# It receives the auth code and goes to the OAuth 2.0 provider to get the subject token. The jwt
# token is then sent to the browser with two cookies with splitting header/payload and signature.
# Another options is to keep the jwt in session and return sessionId to the browser. In either
# case, the csrf token will be send with a separate cookie.
---
# Indicate if the StatelessAuthHandler is enabled or not
enabled: true
# Once Authorization is done, which path the UI is redirected.
redirectUri: https://localhost:3000
# An optional redirect uri if the user deny or cancel the authorization on the Consent page. Default to redirectUri if missing.
denyUri: https://localhost:3000
# Request path for the authorization code handling.
authPath: /authorization
# Request path for the deny authorization handling to remove HttpOnly access-token and other cookies
logoutPath: /logout
# Cookie domain
cookieDomain: localhost
# Cookie path
cookiePath: /
# Login uri, redirect to it once session is expired
cookieTimeoutUri: /
# Cookie secured
cookieSecure: true

```

Both redirectUri and denyUri are https://localhost:3000, and the cookie domain is localhost. 

client.yml

```
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e74
      # client_secret for the authorization code flow. For 2.0.x the secret can be set here. 
      client_secret: f6h1FTI8Q3-7UScPZDzfXA
      # the web server uri that will receive the redirected authorization code
      # redirect_uri: https://ob.lightapi.net/authorization
      # optional scope, default scope in the client registration will be used if not defined.
      # scope:
      # - petstore.r
      # - petstore.w
    # the following section defines uri and parameters for client credentials grant type

```

In the authorization code grant flow configuration, we have commented out the redirect_uri so that the OAuth 2.0 client registration redirect_uri https://localhost:3000 will be used. If we still have this optional configuration, the OAuth 2.0 server will try to match this redirect_uri with the client registered redirect_uri and return an error. 

handler.yml

```
  - path: '/authorization'
    method: 'GET'
    exec:
      - default
  - path: '/logout'
    method: 'GET'
    exec:
      - default

```

We need to ensure that /authorization and /logout are configured in the handler.yml

cors.yml

```
enabled: true
allowedOrigins:
  - https://localhost:3000
  - https://obsignin.lightapi.net
  - https://ob.lightapi.net
allowedMethods:
  - GET
  - POST
  - PUT

```

As you can see, the https://localhost:3000 is in the allowedOrigins. 

### react-client

The react-client application is generated with create-react-app. We need to update a little bit to make sure that it can work with backend APIs and start with HTTPS as it is required by the OAuth 2.0 specification. 

After the code is generated, we need to add the following to the package.json

```
    "start": "react-scripts start",
    "startHttps": "HTTPS=true react-scripts start",
```

In the script section, we added startHttps with HTTPS=true environment variable. We need to use it to start the Nodejs server so that it will listen to HTTPS and port 3000. 

To start the Nodejs server on a terminal.

```
yarn startHttps
```

We also need to set up the proxy configuration in the setupProxy.js

```
const proxy = require('http-proxy-middleware');
const cors = require('cors');

module.exports = function(app) {
  app.use(cors());
  app.use(proxy('/accounts', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/balances', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/beneficiaries', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/direct-debits', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/offers', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/parties', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/products', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/scheduled-payments', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/standing-orders', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/statements', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/transactions', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/authorization', { target: 'https://ob.lightapi.net', secure: false }));
  app.use(proxy('/logout', { target: 'https://ob.lightapi.net', secure: false }));
};

```

As you can see, we have enabled the CORS and also added all the API endpoints to the proxy. The last two endpoints are for the light-spa-4j StatelessAuthHandler. 

There is another place in the react application that needs to be updated. When the react-client is redirecting to the login-view application for the light-oauth2 authentication and authorization, it can pass a redirectUri optional. Since we don't want to switch between https://localhost:3000 and https://ob.lightapi.net constantly, we need to remove the redirectUri from the redirect so that the OAuth 2.0 provider will use the redirectUri provided by the client registration. 

Here is the login function without redirect URL in ResponsiveDrawer.js

```
    const login = () => {
        window.location = "https://obsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e75&user_type=customer&state=1222";
    }

```

If you checked the database script in the [oauth-config](/tutorial/open-banking/client/oauth-config/), you can see the above client_id has a redirect_uri as https://localhost:3000


{{< youtube xeG0Va1UxVY >}}