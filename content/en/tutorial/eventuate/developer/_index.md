---
title: "Developer Guideline"
date: 2018-01-07T09:29:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The environment setup is a little bit complicated in light-eventuate-4j framework and here
are want to give some advices on what is the best practices on setting develop environment
for both light-eventuate-4j developers or services developers as the focuses are different
for these two type of developers.

If you just want to developer microservices on top of light-eventuate-4j framework, things
are simpler. You can follow the instructions in the [service developer tutorial][] to get
everything set up.

If you want to add new features or fix some defects in light-eventuate-4j framework. Or
just want to debug into it to learn more about the framework, there are a lot of components
should be started manually instead of Docker so that you have more control on how it should
be working. Please follow the [eventuate developer tutorial][] for more information. 

[service developer tutorial]: /tutorial/eventuate/developer/service-dev/
[eventuate developer tutorial]: /tutorial/eventuate/developer/eventuate-dev/
