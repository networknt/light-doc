---
title: "Start Consul"
date: 2019-04-30T10:34:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have the [environment parepared][]. In this step, we are going to start the consul server for other subsequent services to register and discover services.

All portal services will be registered to the Consul cluster on the official test/prod environment. To make sure that the local dev environment works the same. We are going to start the consul server locally with a docker-compose in the light-docker repository.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml
```

The above docker-compose starts a container with only one consul node. The docker-compose uses `localnet` as network name for it is a standalone service.  All other services should be using the host network to register to the consul instance. 

In the config, we have set the `"acl_master_token":"the_one_ring"` so we need to setup consul-token to `the_one_ring` in the secret.yml for all services. 

After the docker-compose is up, we can go to the UI to check the services on the browser. 

Put the following URL into the address bar of your browser, and you can see there is one service named consul that is health from the Services tab. 

```
http://localhost:8500
```

As we have added lightapi.net into the /etc/hosts, you can also use the following URL to access the consul UI.

```
http://lightapi.net:8500
```

If you want to learn more about the docker-compose-consul.yml, please visit the source code in the local folder or on GitHub at https://github.com/networknt/light-docker/blob/master/docker-compose-consul.yml

In the next step, we are going to [start the light-oauth2][] services and let them register to the consul instance just started. 

[environment parepared]: /tutorial/portal/local-router/prepare-environment/
[start the light-oauth2]: /tutorial/portal/local-router/light-oauth2/
