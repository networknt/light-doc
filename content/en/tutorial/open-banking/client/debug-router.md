---
title: "Debug Router"
date: 2020-02-15T14:31:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-router instance serves the Single Page Application and acts as a proxy and load balancer to access the APIs behind it. From the previous [login-view][], we have seen the complicated interactions between the Single Page Application,  Login View of OAuth 2.0 provider, and the StatelessAuthHandler wired in the light-router chain. This puts the light-router as a central point in the entire application. To solve issues with the UI, and microservices, it is easier to debug into the router instance. 

There are two ways to debug light-docker. 


### Local Standalone

As the light-4j application is just a POJO without any container, you can [debug locally][] just like a simple Java application with the entry point as com.networknt.server.Server. If you are using the externalized configuration folder, then you can add an option as -Dlight-4j-config-dir=xxx

The config folder for the standalone debug is located at https://github.com/networknt/light-config-test/tree/master/light-router/local-openbanking/debug

The only file that is different than the normal config folder is the virtual-host.yml file.

```
hosts:
  - domain: obsignin.lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/local-openbanking/obsignin/build
    #base: /obsignin/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: ob.lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/local-openbanking/ob/build
    #base: /ob/build
    transferMinSize: 10245760
    directoryListingEnabled: false
```

Notice that we are using the absolute path for the base to load the two single-page applications. 

{{< youtube BUNjrHRbHZ8 >}}

### Remote Docker Container

Sometimes, the application might work well in standalone mode but doesn't work inside a docker container. In most cases, this is due to the docker network issue. A lot of developers are getting used to the localhost and don't know that the same localhost points to the docker OS loopback address instead of the docker host address. 

To resolve issues in the docker container, you can connect to the application within the docker container to [debug remotely][]. 

The configuration folder for the docker remote debug is located at https://github.com/networknt/light-config-test/tree/master/light-router/local-openbanking/config

In the local-openbanking folder, we have a docker-compose-debug.yml that is loading the container with the debug tag. 

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:debug
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

To create the light-router debug docker image. 

```
cd ~/networknt/light-router
mvn clean install -DskipTests
docker build -t networknt/light-router:debug -f ./docker/Dockerfile-Debug .
```

Once the local-openbanking works, we need to deploy everything to the [test cloud][] so that the demo application can be accessed from the Internet. 

{{< youtube QuG87ByBNaY >}}

[login-view]: /tutorial/open-banking/client/login-view/
[debug locally]: /tutorial/common/debug/idea/
[debug remotely]: /tutorial/common/debug/docker-remote/
[test cloud]: /tutorial/open-banking/client/test-cloud/
