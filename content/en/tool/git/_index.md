---
title: "Git Source Control System"
date: 2018-03-09T16:18:41-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Most our projects are open source and repositories can be found at github.com/networknt organization. 
Some of the internal repositories are hosted on our private git server at https://git.lightapi.net/

Git is a very important tool for us and this section contains several important tricks and tips in
using git. 


After git is installed, you need to initialize it with some global config. 


Here is what I do on my desktop. 


```
git config --global user.name "Steve Hu"
git config --global user.email stevehu@gmail.com
```

To remove a sensitive file from the history, please refer to https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository

- [Remove Big Binary Files from Repo](/tool/git/remove-bigfile/)
 