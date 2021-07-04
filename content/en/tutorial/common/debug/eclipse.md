---
title: "Eclipse"
date: 2017-12-20T17:47:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Utilizing the Eclipse IDE, it is similar to the same debugging process on IntelliJ IDEA.

Here is the steps to create a standalone application in Eclipse IDE.

* Click the drop down menu on the *Run* button and select "Run Configurations"
* Click + on the top left corner to add a new Configuration.
* In the side menu bar double click on "Java Application"
* Name your application to Server
* In Main Class input, type com.networknt.server.Server
* Click apply to close the popup window.
* Click Run again and select debug
* Pick the application configuration you just created.

>Using eclipse, you have to run as debug in order to see logs and breakpoints in console.


<!-- Utilizing HTML tag here due to inline size change required -->
<img src="/images/eclipse-ide-debug.png" alt="eclipse ide debugging" style="width:800px;"/>




Now the server is up and running in the debug mode. You can debug into the framework
code or set a break point in framework code. To pick up the framework class to debug, 
click External Libraries on the Project tree and click the class name. IDEA will
automatically decompile the class. You can import the source code of that particular
version of framework source code from [Release](https://github.com/networknt/light-4j/releases)

##### Return to [Common Tutorial Home Page](/tutorial/common)


