---
title: "Light Bot Tutorial"
date: 2017-12-30T11:01:56-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 60
weight: 60
aliases: []
toc: false
draft: false
reviewed: true
---

This is the tutorial to learn how to run the light-bot for networknt projects and how to leverage it for your organizations and repositories. As it is using config module from light-4j, the config files can be externalized to somewhere in the host filesystem.
You can deploy it in one of your DevOps VMs and set it up to run automatically for developing build
and test. Other tasks are triggered manually and only used occasionally.

* Clone and [build light-bot][] project and command line tool with Gradle
* Set up [develop][] branch build and test on your local computer
* Set up [devops][] sever for develop branch build automatically
* Build [light-portal local][] develop branch using cloud eventuate services
* How to use [regex-replace][] to replace repeated string/text across all repositories


[develop]: /tutorial/bot/local-develop/
[devops]: /tutorial/bot/devops-develop/
[light-portal local]: /tutorial/bot/light-portal-local/
[build light-bot]: /tutorial/bot/build-light-bot/
[regex-replace]: /tutorial/bot/regex-replace/
