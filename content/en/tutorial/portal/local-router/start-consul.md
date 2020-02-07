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

In the previous step, we have updated the /etc/hosts on the desktop to map the local IP address to the lightapi.net. With the latest version of the Docker, it might not correctly use the Docker HOST /etc/hosts to resolve the DNS. To ensure it works on all environments, we are going to update the docker-compose-consul.yml to add the mapping to the consul service. 

Line 14 and 15 below should be updated with your local IP address. 

https://github.com/networknt/light-docker/blob/master/docker-compose-consul.yml#L14

The above link is pointing to GitHub.com, but you need to update the local file that you have checked out in `~/networknt/light-docker` folder. Once the update is done locally, run the following command to start the docker-compose. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

The above docker-compose starts a container with only one consul node with `localnet` as the network name.  All other services should be using the host network to register to the consul instance. 

In the config, we have set the `"acl_master_token":"the_one_ring"` so we need to setup consul-token to `the_one_ring` in the secret.yml for all services. If you are using the light-4j 2.0.x and above, you don't need the separate secret.yml file for the consulToken, but just put it into the consul.yml instead. All config files support encryption since version 2.0.0 

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
