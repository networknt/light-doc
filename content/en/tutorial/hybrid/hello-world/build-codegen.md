---
title: "Build Light-codegen"
date: 2019-04-16T16:17:44-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous [workspace][] step, we have cloned related repositories to the networknt workspace in the home directory. 

In this step, we are going to build the light-codegen so that we can use the command line to scaffold projects. 

Maven is the build tool for light-codegen, and we have the maven wrapper included in the repository. If you don't have Maven installed on your local computer, you can use the `mvnw` or `mvnw.cmd` to build the project. 

```
cd ~/networknt/light-codegen
./mvnw clean install
```

If you have Maven installed locally, you can use the `mvn` command to build the light-codegen.

```
cd ~/networknt/light-codegen
mvn clean install
```

If there is no error in the build process, the command line CLI tool codegen-cli.jar is ready to use in light-codegen/codegen-cli/target folder. 

In the next step, we are going to [generate a server][] project to host the two services we are going to build later. 


[workspace]: /tutorial/hybrid/hello-world/workspace/
[generate a server]: /tutorial/hybrid/hello-world/generate-server/
