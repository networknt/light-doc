---
title: "OAuth2 Console"
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

In the previous step, we have [started the light-router][] instance for the local portal. It listens 443 for the host lightapi.net

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

A browser tab will be automatically started and any code change in the React SPA will be hot loaded by the browser. 


[started the light-router]: /tutorial/portal/local-router/light-router/
