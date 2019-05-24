---
title: "Intellij Idea"
date: 2017-12-20T17:47:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


I am using Intellij Idea but Eclipse should be very similar as the generated project is
standard Maven project. To debug your code, you need to start the server.

Here is the steps to create a standalone application in Idea.

* Click Run and select Edit Configurations...
* Click + on the top left corner to add a new Configuration.
* In the dropdown select application
* Name your application to Server
* In Main Class input, type com.networknt.server.Server
* Click apply to close the popup window.
* Click Run again and select debug
* Pick the application configuration you just created.

Now the server is up and running in the debug mode. You can debug into the framework
code or set a break point in framework code. To pick up the framework class to debug, 
click External Libraries on the Project tree and click the class name. Idea will
automatically decompile the class. You can import the source code of that particular
version of framework source code from [Release](https://github.com/networknt/light-4j/releases)


