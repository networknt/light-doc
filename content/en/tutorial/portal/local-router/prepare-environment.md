---
title: "Prepare Environment"
date: 2019-04-30T08:55:57-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before starting portal services locally, we need to make sure that docker is installed. This tutorial is based on Ubuntu Linux, and I am sure that Mac OS should work the same. However, I am not sure that it will work on Windows. It would be great if someone can help to test it on the Windows environment. 

### Check out

We have docker-compose and config files checked into some repositories on the GitHub.com, and we need to check out these repositories. If you want to make a code change for one of the light-portal services, then you need to check out more repositories and build docker images locally. 

As usual, I am using networknt under my home directory as the workspace. 

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
git clone https://github.com/networknt/light-config-test.git
git clone https://github.com/networknt/light-portal.git
```

light-docker contains most commonly used docker compose files and their config files. 
light-config-test contains vary config files and script for daily development and test/prod deployment.
light-portal contains the view that is a React SPA to access the services through the router.


### /etc/hosts

When we use the consul for service discovery, we need to use docker host network, and all the services will be bound to the same IP/host with different ports. In order to simplify the service reference and later we can use the domain name to access the SPA from the browser, let's create a hostname lightapi.net in the /etc/hosts to point to the network address. 

First, use the following command to find out your local network IP address.

```
ifconfig
```

The local IP address is 192.168.1.144 on my desktop. You cannot use the 127.0.0.1 or localhost as they mean local address within a Docker container. 

Now, let's add a line into the /etc/hosts file with `sudo vi /etc/hosts`

```
192.168.1.144   lightapi.net
```

Note that your local IP address might be different than mine. Please update the above line accordingly. 

In the next step, we are going to [start a consul][] container to mimic a consul cluster on a test/prod environment. 

[start a consul]: /tutorial/portal/local-router/start-consul/



