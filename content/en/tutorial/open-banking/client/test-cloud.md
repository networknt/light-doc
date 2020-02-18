---
title: "Open Banking Demo on the Test Cloud"
date: 2020-02-09T15:42:03-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In previous steps, we have tested the demo application on the local. Now, it is time to deploy the application to the test cloud. 

The two built sites are in the following folder in the light-config-test repository. 

https://github.com/networknt/light-config-test/tree/master/light-router/test-openbanking

The light-router docker-compose file looks like the following. 

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    # networks:
    # - localnet
    ports:
    - 8443:8443
    volumes:
    - ./config:/config
    - ./obsignin/build:/obsignin/build
    - ./ob/build:/ob/build
    network_mode: host
```

The configuration for the light-router is located at 

https://github.com/networknt/light-config-test/tree/master/light-router/test-openbanking/config

To start the server on the sandbox server. 

```
cd ~/networknt/light-config-test
git pull origin master
docker-compose up -d
```

After we get the router instance up and running, we can walk through the application. Here is a video and everything can access the application at https://ob.lightapi.net

If you are testing the application and encounter any issue, please feel free to ask questions at https://gitter.im/networknt/light-4j.


