---
title: "Portal View"
date: 2019-05-01T16:35:49-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [started the light-router][] instance for the local portal. It listens to 443 for the host lightapi.net

In this step, we are going to explore the lightapi.net UI implementation with a focus on the light-oauth2 view.

For development, we are going to use a proxy to access APIs through light-router. The light-portal view can be accessed from light-portal/view folder. 

https://github.com/networknt/light-portal/tree/master/view


### proxy

In the src folder, there is a setupProxy.js which contains all the paths for APIs and the target host mapping. 

```
const proxy = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(proxy('/portal/command', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/portal/query', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/client', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/code', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/key', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/refresh_token', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/service', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/token', { target: 'https://lightapi.net', secure: false }));
  app.use(proxy('/oauth2/user', { target: 'https://lightapi.net', secure: false }));
};

```

### Start

If you haven't run the `yarn install` in the view folder, please do so first. 

To start the testing Nodejs server.

```
yarn start
```

A browser tab will be automatically started, and any code change in the React SPA will be hot loaded by the browser. 

The SPA is connecting to the proxy on port 3000 of the localhost, and all API access is proxied to the https://lightapi.net with the path prefix defined in the above setupProxy.js

You can now click the OAuth 2 menu on the browser to access the light-oauth2 view. When you change the view source code, it will be reflected immediately on the browser. 

In the next step, we are going to build the view SPA and [deploy][] it to the light-router local instance. When the light-router serves the view, there is no need to have any proxy and the performance of the SPA application will be much better due to the CORS is skipped.  


[started the light-router]: /tutorial/portal/local-router/light-router/
[deploy]: /tutorial/portal/local-router/deploy-view/
