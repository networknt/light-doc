---
title: "Preparation"
date: 2017-11-29T16:42:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

2017-11-29T10:32:34-05:00

All specifications and code of the services are on github.com but we are going to
redo it again by following the steps in the tutorial. Let's first create a
workspace. I have created a directory named networknt under user directory.

Checkout related projects into your local workspace.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-oauth2.git
git clone https://github.com/networknt/light-docker.git

```

As we are going to regenerate API A, B, C and D, let's rename ms_chain folder from
light-example-4j so that we can compare with the standard implementation if something
is not working.

```
cd ~/networknt/light-example-4j/rest/swagger
mv ms_chain ms_chain.bak
cd ~/networknt
```

In the next step, we will review the specifications for these APIs. 

