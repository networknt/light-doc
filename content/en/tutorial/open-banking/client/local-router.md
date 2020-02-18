---
title: "Light Router for Open Banking"
date: 2020-02-10T11:20:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we started the [light-oauth2][] services locally. In this section, we are going to start the light-router instance to act as a consumer proxy and host the two single-page applications as a virtual host. 

After we completed the [cookie][] code and tested it, we need to deploy the react application build to the [local-openbanking][] folder to host it by the light-router. For each secured client application, we need to have a light-router instance to handle Internet access, so we cannot reuse the light-portal instance for the portal server anymore. We are going to start another instance on to host https://ob.lightapi.net and https://obsignin.lightapi.net and it is listening to 8443. 

Here is the docker-compose.yml file in the [local-openbanking][]

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    ports:
    - 8443:8443
    extra_hosts:
    - "obsignin.lightapi.net:192.168.1.144"
    - "ob.lightapi.net:192.168.1.144"
    volumes:
    - ./config:/config
    - ./obsignin/build:/obsignin/build
    - ./ob/build:/ob/build
    network_mode: host
```

In order to redirect from 443 to 8443, we need to update the iptables based on this [redirect 443 to 8443][] tutorial.

On my desktop, we just need to do the following. 

```
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -o lo -j REDIRECT --to-port 8443
```

Once the above is done, can access the demo application at https://ob.lightapi.net site. 


[cookie]: /tutorial/open-banking/client/cookie/
[local-openbanking]: https://github.com/networknt/light-config-test/tree/master/light-router/local-openbanking
[test-openbanking]: https://github.com/networknt/light-config-test/tree/master/light-router/test-openbanking
[redirect 443 to 8443]: /tutorial/security/port443/
[light-oauth2]: /tutorial/open-banking/client/oauth-config/
