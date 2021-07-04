---
title: "Prepare Environment"
date: 2018-01-07T16:25:00-05:00
description: ""
categories: []
keywords: []
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

Assuming that light-eventuate-4j platform is up and running, let's check out the light-example-4j project and rename the todo-list folder in eventuate so that we can
recreate it in this tutorial.

As with other tutorials, we are going to use networknt under user's home directory as our workspace. 

The default configuration in the source code uses remote Kafka and MySQL hosted in the cloud for developers. We encourage you to set up your own eventuate environment locally once you feel comfortable so that you can learn the process. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j/eventuate
mv todo-list todo-list.bak
``` 

Once all done, the todo-list project in light-example-4j/eventuate contains POJOs, two restful services, and two hybrid services.  

When you are ready, let's go to the next step to [create multi-module maven project][].

[create multi-module maven project]: /tutorial/eventuate/todo-list/project/

