---
title: "Full Stack Developer"
date: 2020-03-02T20:27:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [View React UI developer][] tutorial, we have described how to deploy the dev.lightapi.net and dev.signin.lightapi.net on the test cloud to support UI developers to leverage all services in the cloud without installing anything locally. It is very convenient for UI developers because they only need to start the Nodejs server to develop locally with hot reload, and all the OAuth 2.0 authorization code flow interactions with backend services are handled already. What if we want to debug the interaction with the light-oauth2 from the UI, or we want to update the backend services and test them from the UI. For a full-stack developer, he/she needs to get everything deployed on the local desktop or laptop so that he/she can even work without Internet access. 

In this tutorial, we are going to show users how to deploy the light-oauth2, light-portal, and light-router locally to allow the React app to access backend in debug mode. For me, this is a typical setup on my desktop computer. 

### Environment

We are going to use the following two domains for the light-portal virtual host. These are used on the test cloud, but we can update the /etc/hosts to simulate the local DNS.

```
192.168.1.144   dev.lightapi.net dev.signin.lightapi.net dev.oauth.lightapi.net
```

We are going to use Consul for service discovery. To start the Consul Docker container, use the following commands.


```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

You can access the Consul UI from the browser at `http://localhost:8500/` to check all the registered services after starting the light-oauth2 and light-portal services. 

With Consul up and running, we can start the light-oauth2 services with the following commands.


```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose up -d
```

Now we can start the hybrid-command and hybrid-query servers in debug mode in IDE to test/debug the interactions with the UI. For details, please refer to [debug-both-servers][].

With all backend services started, we need to start the light-router instance to serve the virtual hosts and proxy the calls to the backend services. 

```
cd ~/networknt/light-config-test/light-router/local-portal
docker-compose up -d
```

The light-portal router is listening to 8443 and we need to redirect 443 to 8443 with iptables. 

```
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -o lo -j REDIRECT --to-port 8443
```



### React UI

Start the portal view application in HTTPS. 

```
cd ~/networknt/light-portal/view
yarn startHttps
```




[View React UI developer]: /tutorial/portal/developer/react-ui/
[debug-both-servers]: /tutorial/portal/developer/debug-both-servers/
