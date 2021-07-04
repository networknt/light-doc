---
title: "Add Jars to Classpath"
date: 2018-12-02T22:34:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When debugging light-hybrid server, it is necessary to add all the jar files from the services target directory to the server classpath in the IDE. 

To do that, follow the steps below. 

* Click File/Project Structure and click the Modules on the Project Settings menu from the popup window. 
* On the right side of the panel, click Dependencies tab and you will see all the dependencies for the project. 
* Click the "+" sign on the right to add Jars or Directories. 
* Pick up all the jars files from each service to add. 

Once this is done, you can debug the server just like regular light-4j service. 


{{< youtube Kpv0TVESKDA >}}

