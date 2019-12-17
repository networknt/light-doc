---
title: "Open Banking SPA Client"
date: 2019-12-14T21:14:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [router][] step, we have exposed the services to the Internet through https://ob.lightapi.net subdomain. Eventually, the services will be consumed by other services, single page applications or mobile applications. In this step, we are going to build a consumer application using React and deploy it to the light-router instance on the portal as another virtual host. 

In the previous steps, we are using long-lived tokens to access services. With a single page application, we can also close the entire loop of OAuth 2.0 authorization code flow. 

To start a React application, we need to create a repo on the github under open-banking organization. Then clone the repo to the open-banking folder under home directory. 

```
git clone git@github.com:open-banking/react-client.git
npx create-react-app react-client
yarn start
```

A browser tab will be started automatically after the node server is up with the SPA. Let's shutdown the server and check in the generated code. 

```
git add .
git commit -m "initial checkin"
git push origin master
```






[router]: /tutorial/open-banking/router/
