---
title: "Dependency Build"
date: 2018-07-30T06:57:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For developers who are working on the Light platform, they have to work on the develop branch which is not released to the maven central yet. When they are working on a repo which depends on other repositories, Maven will try to find these dependencies in the local .m2 directory and will report an error if it cannot locate them. 

For experienced developers, they can figure out which module is in the dependency tree and clone/build these modules individually until the current project can be built. However, this takes a lot of time, and it is a repetitive task. To streamline the process, we have introduced a light-bot config to checkout and build all dependencies for the networknt organization. 

As it is a light-bot task, we have to clone/build light-bot first, and this is only a one time task. 

### Build light-bot

Please refer to [build light-bot][] tutorial for details. 

### Build all repos

This task will check out all related repositories from the networknt organization and build them all together without triggering any test cases. Some of the test cases require databases and Kafka to be running, but for dependency build, they are not necessary. 

Assume that you are using networknt as your working directory.

```
cd ~/networknt/light-config-test/light-bot/develop-build/build-all
./run.sh
```

The above command will build all dependencies and put them into the local .m2 directory for the development build. 





[build light-bot]: /tutorial/bot/build-light-bot/

