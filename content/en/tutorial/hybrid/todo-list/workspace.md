---
title: "Todo-list Workspace"
date: 2019-04-09T10:33:18-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

All specifications and code of the services are on the GitHub.com, but we are going to redo it again by following the steps in the tutorial. Let’s first create a workspace. I have created a directory named networknt under user's home directory.

Check out related repositories.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
```

Let's go to the light-codegen folder to build the project.

```
cd light-codegen
./mvnw clean install
```

If you have Maven installed locally, you can use `mvn clean install` instead. 

The todo-list example are located in the light-example-4j/eventuate/todo-list folder. We are going to regenerate hybrid-command and hybrid-query services. So let's rename the existing folder so that we can compare the existing code with the newly generated code if necessary. 

```
cd ~/networknt/light-example-4j/eventuate/todo-list/
mv hybrid-command hybrid-command.bak
mv hybrid-query hybrid-query.bak
```

With the workspace ready, we are going to [generate the command service][] in the next step. 


[generate the command service]: /tutorial/hybrid/todo-list/generate-command/

