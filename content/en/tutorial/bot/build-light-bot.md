---
title: "Clone and Build Light Bot"
date: 2018-03-31T07:25:01-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

light-bot is a tool built on top of the light-4j framework. Before using it, we have to clone and build it locally in our workspace. 

To make it easier, we use networknt under user home directory for our workspace. Please follow the steps below to get light-bot built locally. If you have networknt workspace already, you do not need to create the directory. 

```
cd ~
mkdir networknt
cd networknt
git clone https://github.com/networknt/light-bot.git
cd light-bot
./gradlew build
```

Once the build is completed, you can find a bot-cli-fat-1.0.jar file in `~/networknt/light-bot/bot-cli/build/libs` folder. This jar file contains the command line class to execute light-bot tasks. 

As you can see this project is built with Gradle as we are trying to support both Maven and Gradle in the light-codegen for API projects. We are using this project to test Gradle Kotlin DSL. 

As we have Gradle wrapper checked into the project, you don't need to install Gradle on your computer to build the project. The only dependency is Java JDK. The first time build will take a while as Gradle 4.4.1 and Kotlin DSL need to be downloaded. 
