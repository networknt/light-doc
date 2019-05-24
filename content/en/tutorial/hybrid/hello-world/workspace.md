---
title: "Prepare Hybrid Hello World Workspace"
date: 2019-04-04T10:24:35-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this section, we are going to prepare the local environment and workspace for the rest of the tutorial. 

### JDK8

We are going to use the master branch of each repository for the tutorial, so JDK 8 is required to be installed on your computer. If you have another version of JDK installed already, the [SDKMAN][] is a great tool to allow you to install multiple versions of JDK and switch between them. 

### Workspace

For this tutorial, we are going to use the `networknt` under the user's home directory for workspace. All related repositories will be cloned into this folder. 

Assume that you have git command line installed on your computer, clone the following repositories. 

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
```

* The light-codegen is the command line tool to scaffold the server project and service1 and service2 projects. 
* The light-example-4j contains the source code for the tutorial. We are going to remove the folder and start over again. 
* The model-config folder contains the specification files for service1 and service2 as inputs for the light-codegen. 

The source code for this tutorial is located in the light-example-4j repository under hybrid/hello-world folder. Before we move to the next step, let's change the hello-world directory to hello-world-bak so that you can rebuilt all project again. The backup folder can be used for comparison if you miss any step or unsure if you follow the tutorial correctly. 

```
cd ~/networknt
mv light-example-4j/hybrid/hello-world light-example-4j/hybrid/hello-world-bak
```

After repositories are cloned to the local workspace, let's go to the next step to [build light-codegen][]. 


[SDKMAN]: /tool/sdk/
[build light-codegen]: /tutorial/hybrid/hello-world/build-codegen/



