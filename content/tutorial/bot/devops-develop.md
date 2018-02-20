---
title: "Devops Develop"
date: 2018-02-19T10:39:54-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In this tutorial, we are going to setup an automatic build process on our devops VM running
in a data center.

### Connection

First let's follow the [ssh] tool tutorial to upload public key to the devops server so that
we don't need to type password every time we want to log into the server.

### Environment

We need to ensure that the following software packages are installed on the server.

* Docker
* JDK
* [Maven][]
* Git

DOCKER_HOST_IP needs to be setup to your real IP address on the server. It is easier to just
put it into .bashrc file at the end. 

```
DOCKER_HOST_IP=xx.xxx.xxx.xx
```

### Checkout Repos

Let's clone all the repositories we need into networknt folder.

Before you can clone repositories from github.com, please generate and upload you key to 
github userprofile if you are using two factor authentication.

The following repos need to be cloned. You can use the ssh protocol instead of git if you
want to try it yourself.

```
git clone git@github.com:networknt/light-bot.git
git clone git@github.com:networknt/light-docker.git
```

### Dependent Services

In order to run the test cases (both unit and integration), we need to start several services
in docker-compose. On this devops server, we are going to leave them running all the time. 

Please refer to [local-develop][] for more info on how to start them. 

### Build and Run light-bot

Please refer to [local-develop][] for more info on how to build light-bot and run it manually
from the command line. 

### Customized Config


### Setup Email

### Setup Cron Job



[ssh]: /tool/ssh/
[Maven]: /tool/maven/
[local-develop]: /tutorial/bot/local-develop/
 