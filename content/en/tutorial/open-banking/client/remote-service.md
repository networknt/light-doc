---
title: "Leverage Remote Service"
date: 2020-02-25T20:32:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Until now, we are using the local Consul server, local light-oauth2 services, and local open-banking services for the react-client development. If I am a developer who is working on both front end and backend, this is OK as both might be changing at the same time. By deploying everything on the local desktop with Docker, we can easily make a change and redeploy the backend services. But what if I am just a front-end React developer, I don't need to make any changes to the backend, and all my focus will be limited in the React SPA on the https://localhost:3000. Is it possible to just start one Nodejs instance to support the local React development and leverage the services in the test cloud? Let's explore it. 

On the test cloud, we have a three nodes Consul cluster, the light-oauth2 services, and the open-banking services. Everything is connected from a light-router instance that is deployed to the sandbox VM. What we need to do is to proxy the API request from the https://localhost:3000 to the https://ob.lightapi.net on the test cloud. When the website is redirected to the login page, we just need to leverage the remote site, which is hosted with the light-router instance on the sandbox to serve the login page from https://obsignin.lightapi.net on the test cloud.

The light-router statelessAuth.yml configuration has redirectUri and denyUri and cookie domain pointing to ob.lightapi.net, but the application that is working locally has a domain of localhost. We need to set up a local light-router instance to use the domain ob.lightapi.net to leverage the test cloud light-router instance.

### react-client

Here are some of the updates for the react-client to start the server with a custom domain and port 443. 

First, we need to create a .env file in the root directory of the react-client. 

```
HOST=ob.lightapi.net
PORT=443
```

This will ensure that the server starts with domain ob.lightapi.net and listens to port 443. To start the server with port 443 on Linux, you have to use Sudo. 


```
sudo yarn startHttps
```

Instead of opening a browser tab automatically, we need to manually type the URL `https://ob.lightapi.net` in the address bar to access the site. You need to accept the certificate risk as we are using a self-signed certificate. 

We also got an error when loading the page. 

```
SecurityError: Failed to construct 'WebSocket': An insecure WebSocket connection may not be initiated from a page loaded over HTTPS.
```

If you encounter the above issue, you need to update the webpackHotDevClient.js in the `node_modules/react-dev-utils` folder. Find line 62, change the protocol. 

```
protocol: window.location.protocol === 'https:' ? 'wss' : 'ws',
```

This is a bug in the create-react-app, and it should have been fixed when you are using this tutorial. For more information, please check https://github.com/facebook/create-react-app/pull/8079

After you restart the server, the website should be loaded. You might need to clean the DNS and hard reset and reload the two sites by following the [node-env](/tutorial/open-banking/client/node_env/) tutorial.

The next step is to change the setupProxy.js in the src folder. Previously, we have all the API access proxy to https://ob.lightapi.net, and now it won't work anymore as ob.lightapi.net is pointing to the local IP address, and we don't have any service deployed to the local. Remember that we have two domains hosted on the test-openbanking light-router? We can switch ob.lightapi.net to obsignin.lightapi.net, which is pointing to the same light-router instance. After the changes, the setupProxy.js looks like. 

```
const proxy = require('http-proxy-middleware');
const cors = require('cors');

module.exports = function(app) {
  app.use(cors());
  app.use(proxy('/accounts', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/balances', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/beneficiaries', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/direct-debits', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/offers', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/parties', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/products', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/scheduled-payments', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/standing-orders', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/statements', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/transactions', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/authorization', { target: 'https://obsignin.lightapi.net', secure: false }));
  app.use(proxy('/logout', { target: 'https://obsignin.lightapi.net', secure: false }));
};

```

Restart the server with `sudo yarn startHttps`, and you can play the SPA to confirm that the react-client works perfectly with all the services deployed remotely. 

### firewall 

As we are listening 443 from the Nodejs server, you might need to remove the 443 redirects to 8443 firewall rule if you have followed previous tutorials. You can find more details on how to do that in the [security tutorial](/tutorial/security/port443/). 


This tutorial is for UI developer only; however, in most of the cases, full-stack developers will need to work on both UI SPA application and the backend services at the same time locally. They need the interactions between the front-end and backend to complete the application. What they need is just an OAuth 2.0 provider so that they don't need to deploy the OAuth 2.0 services locally. In the next tutorial, we are going to explore the [remote light-oauth2](/tutorial/open-banking/client/remote-oauth/) provided by the test cloud. 
