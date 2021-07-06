---
title: "Integration"
date: 2019-01-05T11:16:11-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Most users use the generator in Java command line, docker command line or script which can be easily integrated with any DevOps pipeline. 

For users who want to integrate the generator with the maven build, please refer to [Integrate Code generation with maven build process][]

For users who do not want to clone and build the generator locally, we provide a web service with UI so that you can generate project remotely and then download/build locally. 

codegen-web is the web service built to handle requests from the Internet. The UI is built in React to help users choose the right parameters and model to control how the generator works. This web service is built on top of the light-hybrid-4j framework, and it cannot be started on its own. In order to start the web service, we must first generate a hybrid server and then put the codegen-web jar files into the classpath of the server.

Except Web UI, which is a work in progress, the command line, docker and docker with the script are working stably at the moment. 


[Integrate Code generation with maven build process]: /tutorial/generator/codegen-maven/
