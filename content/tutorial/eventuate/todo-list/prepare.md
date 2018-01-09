---
title: "Prepare Environment"
date: 2018-01-07T16:25:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Assuming that light-eventuate-4j platform is up and running, let's checkout the
light-example-4j project and rename todo-list folder in eventuate so that we can
recreate it in this tutorial.

As other tutorials, we are going to use networknt under user's home directory as
workspace. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j/eventuate
mv todo-list todo-list.bak
``` 

Once all done, the todo-list project in light-example-4j/eventuate will contain POJOs 
, two restful services and two hybrid services.  

When you are ready, let's go to the next to [create multi-module maven project][].

[create multi-module maven project]: /tutorial/eventuate/todo-list/project/
