---
title: "View React UI Developer"
date: 2020-03-02T20:27:22-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the light-oauth2 form authentication [local][] and [test cloud][] deployment, we have set up the light-router to serve the dev.lightapi.net and dev.signin.lightapi.net as virtual hosts. These two tutorials focus on the light-oauth2 implementation with light-router as proxy and host for the two sites. 

In this tutorial, we are going to switch our focus to light-portal UI development. First, we set up a router instance for local.lightapi.net to serve the developer portal for https://localhost:3000 React App running locally. And then, we set up another light-router instance devoauth.lightapi.net as a reverse proxy to the light-oauth2 code, token, and key services deployed on the test4 server in the previous [test cloud][] tutorial. Once it is done, we have all backend services deployed to the test cloud for the portal UI developers. One can go to the networknt/portal-view and start the portal UI Nodejs server with the following command.

```
cd ~/networknt/portal-view/view
HTTPS=true yarn start
or
yarn startHttps
```

Please note that we have HTTPS=true before the `yarn start` or `npm start` so that we can start the create-react-app server with HTTPS instead of HTTP. We need to enable the HTTPS as OAuth 2.0 redirect only allow HTTPS URL, and the secure cookie requires it as well. There is an entry in the package.json so that you can use startHttps if you like. 

For details about the HTTPS=true environment variable, please visit https://facebook.github.io/create-react-app/docs/using-https-in-development

Another way to do that is to modify the package.json as documented in https://medium.com/@danielgwilson/https-and-create-react-app-3a30ed31c904

### light-oauth2

As part of the portal, light-oauth2 is responsible for securing the portal services. There is no need to start another instance of light-oauth2 services. We are going to reuse the instance on the test4 server for the [test cloud][] deployment in the previous tutorial.

As we are going to user https://localhost:3000/authorization for the redirect URI instead of the dev.lightapi.net portal server, we cannot reuse the same client_id/client_secret pair that has https://dev.lightapi.net/authorization as the redirect URI. We need to update the create_mysql.sql for light-oauth2 configuration in https://github.com/networknt/light-config-test/blob/master/light-oauth2/test-cloud/light-oauth2/mysql/create_mysql.sql to add another client. 

To make it simple, we are going to copy the existing client and modify the last digit from 2 to 3. 

```
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e73', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'mobile', 'PetStore Web Server', 'PetStore Web Server that calls PetStore API', 'petstore.r petstore.w', '{"c1": "361", "c2": "67"}', 'https://localhost:3000/authorization', 'admin');
```

This client has the same client_secret, and the redirect_uri is `https://localhost:3000/authorization`, which is the URL for the create-react-app Nodejs server. We will set up the proxy in the react application to proxy the request to devsignin.lightapi.net and local.lightapi.net later on. 

Now, we need to login to the test4 server to check out the latest configuration and restart the docker-compose. 

```
ssh test4
cd ~/light-chain/light-config-test
git pull origin master
cd test4
docker-compose down
docker-compose rm
docker-compose up -d
```

We have updated the SQL script, so we want to start the MySQL docker container from scratch. That is why we are calling `docker-compose rm` to remove the old containers. 

### Signin

Even we are going to deploy another instance of the portal with light-router, we are not going to start another sign-in site. The existing devsignin.lightapi.net is going to be used. The new dev portal local.lightapi.net and devsignin.lightapi.net are going to share the same light-oauth2 instance and other services in the future. 

### light-router

As each light-router instance only supports one client_id, we need to start another router instance local.lightapi.net for the portal UI started with create-react-app on https://localhost:3000 with hot reloading. It helps the portal UI developer to work online without installing any backend servers locally. With the form authentication, developers can work on both authorized routes and unauthorized routes in the react single page application. The light-router instance also proxy the service requests to portal services on the test cloud.

First, we need to set up the DNS on the Cloudflare to point the local.lightapi.net to test4 and devsignin.lightapi.net to our test1 server. 

There are some issues with Cloudflare to redirect the URL to `https://localhost:3000/authorization` after sign in if the traffic goes through the Cloudflare, so we are going to let local.lightapi.net point to the real IP address directly. It means we need to bypass the Cloudflare CDN for the DNS. 

In the test4 folder, we are going to create the config folder add the config files for the local.lightapi.net.

```
cd ~/light-chain/light-config-test/test3
mkdir light-router
```

* client.yml

We need to update the client_id in all places to 

```
client_id: f7d42348-c647-4efb-a52d-4c5787421e73
```

Also, the redirect_uri for the authorization_code flow needs to be changed to 

```
redirect_uri: https://localhost:3000/authorization
```

This URL reflects the create-react-app default port number. Note that we are using HTTPS here as it is required by OAuth 2.0 specification and the light-spa-4j implementation. 

* cors.yml

Here we need to replace lightapi.net with dev.lightapi.net in the allowedOrigins.


```
enabled: true
allowedOrigins:
  - https://devsignin.lightapi.net
  - https://dev.lightapi.net
  - https://dev.maproot.net
  - https://devsignin.maproot.net
  - https://localhost:3000
allowedMethods:
  - GET
  - POST
  - PUT
  - DELETE
```

* statelessAuth.yml

We need to update this config file to change the cookieDomain to localhost from `lightapi.net`

```
# Indicate if the StatelessAuthHandler is enabled or not
enabled: true
# Once Authorization is done, which path the UI is redirected.
redirectUri: https://localhost:3000/#/app/dashboard
# An optional redirect uri if the user deny or cancel the authorization on the Consent page. Default to redirectUri if missing.
denyUri: https://localhost:3000/#/app/dashboard
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

* virtual-host.yml

We need to remove it as there is no need to serve any site for this router instance. The devsignin.lightapi.ent is served by test1 router instance and https://localhost:3000 is served by local Nodejs server. If this file is copied from the test1 router, then it can be removed.

If this file is removed, we need to update the handler.yml to comment out the entry for the handler. 

```
  # - com.networknt.resource.VirtualHostHandler@virtual
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator

and 

defaultHandlers:
  - cors
  # This router is used to proxy services only. 
  # - virtual

```

### Portal services

The hybrid-query and hybrid-command instances on test1, test2 and test3 are shared and can be accessed from the test3 light-portal. 

### Portal View

We need to make some changes to the view single page application in the portal-view repository to leverage the remote services for developing/debugging/testing. 

The Nodejs create-react-app server needs to enable the CORS. Otherwise, when the redirect comes back from the authorization code flow, the Nodejs server won't put the right headers, and the browser will reject the https://localhost:3000/authorization request. 

To do that we need to add the cors module.

```
cd ~/networknt/portal-view/view
yarn add cors
```

After that, we need to update the view/src/setupProxy.js

```
const proxy = require('http-proxy-middleware');
const cors = require('cors');

module.exports = function(app) {
  var corsOptions = {
    origin: 'https://devsignin.lightapi.net',
    credentials: true,
    optionsSuccessStatus: 200 // some legacy browsers (IE11, various SmartTVs) choke on 204
  }
  app.use(cors(corsOptions));
  app.use(proxy('/portal/command', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/portal/query', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/client', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/code', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/key', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/refresh_token', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/service', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/token', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/user', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/authorization', { target: 'https://local.lightapi.net', secure: false }));
  app.use(proxy('/logout', { target: 'https://local.lightapi.net', secure: false }));
};
```

For more information on the cors configuration, please check it out at https://www.npmjs.com/package/cors#configuration-options

For the login redirect, we need to update view/src/components/Header/ProfileMenu.js with the new client_id and redirect_uri.

```
    const signIn = () => {
        // this is the dev url which is the default for local developement.
        var url = "https://devsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e73&user_type=customer&state=1222";
        if(process.env.REACT_APP_SIGNIN_URL) url = process.env.REACT_APP_SIGNIN_URL + "&user_type=customer&state=1222";
        window.location = url;
    }
```

Once the above changes are done, let's start the server in the view folder. 

```
HTTPS=true yarn start
```

We might get an error when loading the page. 

```
SecurityError: Failed to construct 'WebSocket': An insecure WebSocket connection may not be initiated from a page loaded over HTTPS.
```

If you encounter the above issue, you need to update the webpackHotDevClient.js in the `node_modules/react-dev-utils` folder. Find line 62, change the protocol. 

```
protocol: window.location.protocol === 'https:' ? 'wss' : 'ws',
```

This is a bug in the create-react-app, and it should have been fixed when you are using this tutorial. For more information, please check https://github.com/facebook/create-react-app/pull/8079


### Portal View

During the server startup, a browser tab will be opened automatically with the URL https://localhost:3000

Click the upper right Account Icon and click `Sign Up`, you will receive an confirmation email to confirm your registration. By click the link in the email, your account will be confirmed and you can log in with the email and password. 

You can also go the https://dev.lightapi.net or https://dev.maproot.net to register a new user on the test cloud. 

Click the upper right Account Icon and select `Sign In`, and the browser will be redirected to https://devsignin.lightapi.net to ask for userId and password. 

Enter the following credentials you registered.

A consent page will be displayed to ask you if you want the application to access the scopes you have defined in the client registration. If you click the DENY button, you will be logged out immediately and redirect to the Home route. If you select ACCEPT, you have a login session and redirected to the Home route. You will find that the profile menus have shown up. There is a logout menu and you can click it to log out the session. 

From this moment on, you can update the view/src and test the single page application with hot reloader. This setup removes all the extra work that the UI developers need to do and leverage the remote cloud services for all backend portal and light-oauth2 services. Once the portal UI code is done, we can build it and copy the build folder to the light-chain/light-config-test/test1/lightapi folder to deploy to the test cloud.


### MapRoot View

The light-portal services are designed to support mulitple applications at the same time. Along with light-portal view, you can use https://localhost:3000 to develop maproot view. 

The maproot services are deployed along with light-portal services and some of them are shared between the two applications. If you want to work on the maproot view without starting any local service, you can just go to the maproot folder and startHttps

```
cd ~/networknt/maproot/view
yarn startHttps
```

Any user that is registered on dev.lightapi.net or dev.maproot.net can be used to log in. 


[local]: /tutorial/oauth/form-auth-local/
[test cloud]: /tutorial/oauth/test-cloud/
