---
title: "Debugging"
date: 2018-09-01T09:22:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Unlike other frameworks that are based on containers, light-4j is based on a light-weight HTTP core and implemented as POJOs. That means you don't need to use a remote debugger but simply need to debug locally within your IDE. In most of the cases, we are using the unit or integration test cases to trigger the debugger. 

### IntelliJ IDEA

The best IDE for debugging is IntelliJ IDEA, and the free community edition is more than enough. You don't need to open the source code for the dependencies as it can decompile them automatically. Of course, you can load the real source code if you want to see the inline comments. 

The following image shows the "Run/Debug Configurations" window with the main class set as "com.networknt.server.Server". For any other VM Options, you can put it into the VM options section. 

![idea-debug](/images/idea-debug.png)

Here is a video that show several options. 

{{< youtube qW0xIC_A_KI >}}


### Eclipse

To be added.
