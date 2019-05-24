---
title: "Develop Build"
date: 2018-03-07T22:04:57-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "development"
    weight: 20
weight: 20
aliases: []
toc: false
draft: false
---

For developers who are working on the light platform, it is very important to have all related
repositories to be build on local machine and install the jar files into .m2 repository. It is
OK to checkout all the repositories from https://github.com/networknt and build them one by one;
however, it is very time consuming. To our contributor's life easier, we have create a config
in light-config-test repo to utilize light-bot to build necessary repositories in one shot and
sync with remote git server on a daily basis or when it is necessary. 

**Please note**: light-bot is only working with Linux and Mac OS at the moment, porting to Windows
is still working in progress. 

The following are the steps to get is set up.

### Environment

The following software packages need to be installed locally before moving to the next step. 

* Oracle JDK 8 or OpenJDK 8
* Maven 3.5.x
* Git

A github.com account is needed and public key needs to be uploaded to github based on [this doc][].

### Create Workspace

First a workspace needs to be created under your home directory. Most of the time, networknt will
be used. 

```
cd ~
mkdir networknt
``` 

### Checkout Repos and Build light-bot

Next we need to checkout two repositories and build light-bot to bootstrap the process.

```
cd ~/networknt
git clone git@github.com:networknt/light-config-test.git
git clone git@github.com:networknt/light-bot.git
cd light-bot
./gradlew build
```

### Start light-bot develop-build without test

This task will checkout all the related repo or if they are in the developbuild workspace, then only
try to pull from the remote. Once checkout is done, light-bot will build all of them and install the
final jar files to .m2 repository. 

```
cd ~/networknt/light-config-test/light-bot/develop-build/build-all
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -jar ~/networknt/light-bot/bot-cli/build/libs/bot-cli-fat-1.0.jar -t develop-build
```

Wait for several minutes, all the related light platform jar files should be built and installed. 

### Repeat this when necessary

As multiple developers are working on these repositories, there might be several or more checkins 
each day. It is recommended that you run the develop-build task at least everyday. Even better, 
rerun it when you know someone has checked in something that you are depending on. The second time
you run the light-bot develop-build task, it will be much faster as it won't clone every repo again
but just pull the latest code from remote. If there is no change on any of the repos, build will be
skipped automatically. 
 


[this doc]: https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/

