---
title: "Networknt Repositories"
date: 2019-03-24T19:36:16-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-4j is a server that supports middleware handler plugins to enable different [API styles][]. All core components are located in the light-4j repository, and each API style is located in its corresponding repository. For example, light-rest-4j contains all the middleware handlers designed for Swagger 2.0 specification and OpenAPI 3.0 specification. 

For all the repositories that are depending on the light-4j repository, they need to be released at the same time so that these libraries can work together without any issue. It means we have to build these libraries in sequence. 

Currently, we have three release branches for the different type of users. 

* 1.5.x

This is a production branch based on Java 8, and a lot of our users are using the release from this branch for their production services. The latest version is 1.5.32 as of March 2019. To ensure the smooth patch for the production services, we only release defect fixes from this branch, and all releases are backward compatible. 

* master

This is the main development branch, and all development will be focusing on this branch. For defect fixes, some of them will be down ported to the 1.5.x maintenance branch for the patch release. All new features will be merged from feature branches to the master branch and released from this branch as 1.6.x in March 2019. It is based on Java 8 and production ready. If you have a project still in development, please use 1.6.x release. 

* jdk11

As the name indicated, this branch contains the code based on Java 11. We have been working on the Java 11 releases since 2018, but the ecosystem is still missing some dependencies. So far we have light-4j and light-rest-4j released from this branch as part of the 2.0.x release. For most of our customers, they are using Openshift from RedHat and the base image of jdk11 is not ready yet. This blocks them from upgrading to 2.0.x release. If your organization can use official JDK based image published from Docker, then you can try the 2.0.x release. 


Once all dependencies are resolved in the jdk11 branch, we will make the current master branch as 1.6.x and switch jdk11 to master. 

As we have three release branches, all existing light-bot configuration are split into three folders now in the light-config-test repository under light-bot folder.

[API styles]: /style/