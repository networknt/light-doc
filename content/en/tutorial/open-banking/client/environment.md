---
title: "Local and Test Cloud Environments"
date: 2020-02-17T16:30:23-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before we start the react-client, let's introduce the local and test cloud environments first. 


### Local

We are trying our best to mimic the local environment like the test cloud so that we can deploy all components to the test cloud without a lot of changes in configuration. 


For the initial development of the react-client, we are going to use the Nodejs server to leverage the hot reload. Even when we hook up with the light-oauth2 for the authentication and authorization, we can still use the localhost:3000; however, as OAuth 2.0 provider has the limitation that works with only the HTTPS, we can only start the localhost:3000 on the HTTPS port. 

To start the Nodejs server with HTTPS=true, we need to add a line into the generate package.json

```
    "start": "react-scripts start",
    "startHttps": "HTTPS=true react-scripts start",
```

To start the react-client on the terminal.

```
yarn startHttps
```

Once we reach the point that we need to redirect to the login-view application for credentials, we need to update the /etc/hosts to ensure that we can use the same domain names like the test cloud. 

Here is a line in the /etc/hosts that map the two domains to my local desktop address.

```
192.168.1.144   ob.lightapi.net obsignin.lightapi.net
```

The above line will bypass the Internet DNS and use the local IP address for these two domains. When we reach this point, we are not going to use the Nodejs for the hot load anymore. We need to build the react-client and login-view with `yarn build` command on a terminal.

The location of the SPA for the ob.lightapi.net is https://github.com/networknt/light-config-test/tree/master/light-router/local-openbanking/ob/build

The location of the SPA for the obsignin.lightapi.net is https://github.com/networknt/light-config-test/tree/master/light-router/local-openbanking/obsignin/build


On your local, you need to run `yarn build` and copy the build folder to the corresponding light-config-test folder in the workspace and restart the light-router instance to load the latest site. 


### Test

* Consul

On the test cloud, we've already had a Consul cluster running with three nodes. Here is the tutorial on how to set it up.

https://doc.networknt.com/tool/consul/cluster-install/

* light-oauth2

We have all the light-oauth2 services started on the test1 server with the following docker-compose. 

https://github.com/networknt/light-config-test/tree/master/light-oauth2/test-cloud

* open-banking services

We start the open-banking services based on the docker-compose. 

https://github.com/open-banking/light-config-test/tree/master/service

* light-router

We are going to start a light-router instance as a reverse proxy and serve the https://ob.lightapi.net and https://obsignin.lightapi.net

https://github.com/networknt/light-config-test/tree/master/light-router/test-openbanking

In the next step, we are going to walk through the react-client with a hard-coded [long-lived token][].


[long-lived token]: /tutorial/open-banking/client/long-live/
