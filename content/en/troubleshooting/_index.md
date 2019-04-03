---
title: "Troubleshoot"
date: 2018-04-22T19:56:07-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "troubleshooting"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

Frequently asked questions and known issues, recent workarounds pulled from the [gitter][] forums and [reddit] discussions. It also contains information that helps developers to debug applications if something abnormal happens. 


### Debug

As we are dealing with POJO in Java SE, you can debug your entire application in IDE without any remote connection. If you have two or more services interact each other, you can load all of them into the IDE and debug from one service to another. Here are some tutorials for IntelliJ Idea and Eclipse. 

* [Debug with IntelliJ Idea][]
* [Debug with Eclipse][]


### Known Issues

* [Request with French character in body to light-router with http2 to backend with early version of light-4j service][]


[gitter]: https://gitter.im/networknt/light-4j
[reddit]: https://www.reddit.com/r/lightapi/
[Debug with IntelliJ Idea]: /tutorial/common/debug/idea/
[Debug with Eclipse]: /tutorial/common/debug/eclipse/
[Request with French character in body to light-router with http2 to backend with early version of light-4j service]: /troubleshooting/router-http2-to-older-version-service/

