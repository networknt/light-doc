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
mvn clean verify
```

Once the build is completed, you can find a bot-cli.jar file in `~/networknt/light-bot/bot-cli/target` folder. This jar file contains the command line class to execute light-bot tasks. 

light-bot uses Maven for all modules and CLI packaging. Install JDK 25 or newer
and Maven 3.6.3 or newer before building. The CLI also requires Java 25 or
newer at runtime. Maven Enforcer checks the build requirements.

To install module artifacts locally, run `mvn clean install`. Configure tasks
with an external directory and pass `-Dlight-4j-config-dir=/path/to/config`
before `-jar`; the bundled configurations are examples, not complete defaults.
