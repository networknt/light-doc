---
title: "View Dev Environment"
date: 2019-07-29T13:01:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the light-oauth2 form authentication [local][] and [test cloud][] deployment, we have set up the light-router to serve the lightapi.net and signin.lightapi.net as a virtual host. These two tutorials focus on the light-oauth2 implementation with light-router as proxy and host for the two sites. 

In this tutorial, we are going to switch our focus to light-portal UI development.  First,  we set up a router instance for dev.lightapi.net to serve the developer portal as well as a proxy to the light-oauth2 instances deployed on the test1 server in the previous tutorials. Once it is done, we have all backend services deployed to the test cloud for the portal UI developers. One can go to the light-portal/view and start the portal UI Nodejs server with the following command.

```
HTTPS=true npm start
```

Please note that we have HTTPS=true before the `npm start` so that we can start the create-react-app server with HTTPS instead of HTTP. We need to enable the HTTPS as OAuth 2.0 redirect only allow HTTPS URL, and the secure cookie requires it as well. 

For details about the HTTPS=true environment variable, please visit https://facebook.github.io/create-react-app/docs/using-https-in-development

Another way to do that is to modify the package.json as documented in https://medium.com/@danielgwilson/https-and-create-react-app-3a30ed31c904


### light-oauth2

As part of the portal, light-oauth2 is responsible for securing the portal services. There is no need to start another instance of light-oauth2 services. We are going to reuse the instance on the test1 server for the [test cloud][] deployment in the previous tutorial.

As we are going to start another light-router instance for the dev.lightapi.net portal server, we cannot reuse the same client_id/client_secret pair. We need to update the create_mysql.sql for light-oauth2 configuration in https://github.com/networknt/light-config-test/blob/master/light-oauth2/test-cloud/light-oauth2/mysql/create_mysql.sql to add another client. 

To make it simple, we are going to copy the existing client and modify the last digit from 2 to 3. 

```
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e73', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'mobile', 'PetStore Web Server', 'PetStore Web Server that calls PetStore API', 'petstore.r petstore.w', '{"c1": "361", "c2": "67"}', 'https://localhost:3000/authorization', 'admin');
```

This client has the same client_secret, and the redirect_uri is `https://localhost:3000/authorization` which is the URL for the create-react-app. We will set up the proxy in the react application to proxy the request to dev.lightapi.net later on. 

Now, we need to login to test1 server to check out the latest configuration and restart the docker-compose. 

```
ssh test1
cd ~/networknt/light-config-test
git pull origin master
cd light-oauth2/test-cloud
docker-compose down
docker-compose rm
docker-compose up -d
```

We have updated the SQL script, so we want to start the MySQL docker container from scratch. That is why we are calling `docker-compose rm` to remove the old containers. 

### Signin

Even we are going to deploy another instance of the portal with light-router, we are not going to start another sign in site. The existing signin.lightapi.net is going to be used. The new dev portal dev.lightapi.net and signin.lightapi.net are going to share the same light-oauth2 instance and other services in the future. 

### light-router

As each light-router instance only supports one client_id, we need to start another router instance dev.lightapi.net for the portal UI started with create-react-app on localhost:3000 with hot reloading. It helps the portal UI developer to work online without installing any backend servers. With the form authentication, developers can work on both authorized routes and unauthorized routes in the react single page application. 

First, we need to set up the DNS on the Cloudflare to point the dev.lightapi.net to our test2 server with the real IP 38.113.162.52. 

There are some issues with Cloudflare to redirect the URL to `https://localhost:3000/authorization` after sign in if the traffic goes through the Cloudflare, so we are going to let dev.lightapi.net point to the real IP address directly. It means we need to bypass the Cloudflare CDN for the DNS. 

In the test-portal folder, we are going to copy the config folder to dev and modify the config files in dev folder to the dev.lightapi.net.

```
cd ~/networknt/light-config-test/light-router/test-portal
cp -r config dev
```

* client.yml

We need to update the three client_id to 

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
  - https://signin.lightapi.net
  - https://dev.lightapi.net
  - https://localhost:3000
allowedMethods:
  - GET
  - POST
  - PUT
```

* statelessAuth.yml

We need to update this config file to change the cookieDomain to localhost from `lightapi.net`

```
# Indicate if the StatelessAuthHandler is enabled or not
enabled: true
# Once Authorization is done, which path the UI is redirected.
redirectUri: /
# Request path for the authorization code handling.
authPath: /authorization
# Cookie domain
cookieDomain: localhost
# Cookie path
cookiePath: /
# Cookie maxAge
cookieMaxAge: 600
# Login uri, redirect to it once session is expired
cookieTimeoutUri: /
# Cookie secured
cookieSecure: true
```

* virtual-host.yml

We need to remove all domains and add dev.lightapi.net, and it shares the same base as lightapi.net

```
hosts:
  - domain: dev.lightapi.net
    path: /
    #base: /home/steve/networknt/light-config-test/light-router/local-portal/lightapi/build
    base: /lightapi/build
    transferMinSize: 10245760
    directoryListingEnabled: false
```


### Portal View

We need to make some changes to the view single page application in the light-portal repository to leverage the remote services for developing/debugging/testing. 

The Nodejs create-react-app server needs to enable the CORS. Otherwise, when the redirect comes back from the authorization code flow, the Nodejs server won't put the right headers, and the browser will reject the https://localhost:3000/authorization request. 

To do that we need to add the cors module.

```
cd ~/networknt/light-portal/view
yarn add cors
```

After that, we need to update the view/src/setupProxy.js

```
const proxy = require('http-proxy-middleware');
const cors = require('cors');

module.exports = function(app) {
  app.use(cors());
  app.use(proxy('/portal/command', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/portal/query', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/client', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/code', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/key', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/refresh_token', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/service', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/token', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/oauth2/user', { target: 'https://dev.lightapi.net', secure: false }));
  app.use(proxy('/authorization', { target: 'https://dev.lightapi.net', secure: false }));
};
```

Compare with the copied file; we have changed target to https://dev.lightapi.net for all routes. Also, we add the line `app.use(cors());`

As a temporary solution, we need to update view/src/components/ResponsiveDrawer.js with the new client_id and redirect_uri.

```
window.location = "https://signin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e73&user_type=employee&redirect_uri=https://localhost:3000/authorization&state=1222";
```

Once the above changes are done, let's start the server in the view folder. 

```
HTTPS=true npm start
```

### Test

During the server startup, a browser tab will be opened automatically with the URL https://localhost:3000

Click the upper right login icon, and the browser will be redirected to https://signin.lightapi.net to ask for userId and password. 

Enter the following credentials.

```
userId: admin
password: 123456
```

And the browser will be redirected back to https://localhost:3000

From this moment on, you can update the view/src and test the single page application with hot reloader. This setup removes all the extra work that the UI developers need to do and leverage the remote cloud services for all backend portal and light-oauth2 services. Once the portal UI code is done, we can build it and copy the build folder to the light-config-test/light-router/test-portal/lightapi folder.


[local]: /tutorial/oauth/form-auth-local/
[test cloud]: /tutorial/oauth/test-cloud/
