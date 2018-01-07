---
title: "Create Multi-Module Maven Project"
date: 2018-01-07T16:28:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Let's create a todo-list folder and copy the pom.xml from the existing project.
```
cd ~/networknt/light-example-4j/eventuate
mkdir todo-list
cd todo-list
cp ../todo-list.bak/pom.xml .

```

Now let's copy common, command and query modules to the todo-list folder from existing
one.

- common contains events and model
- command contains command definitions and command side service
- query contains query side service
 

```
cd ~/networknt/light-example-4j/eventuate/todo-list
cp -r ../todo-list.bak/common .
cp -r ../todo-list.bak/command .
cp -r ../todo-list.bak/query .

```

Now let's open pom.xml to remove other modules and only leave common, command and query:

```
    <modules>
        <module>common</module>
        <module>command</module>
        <module>query</module>
    </modules>

```

Let's run the maven build to make sure these three modules can be built.

```
cd ~/networknt/light-example-4j/eventuate/todo-list
mvn clean install
```

