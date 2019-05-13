---
title: "Preparation"
date: 2018-03-11T20:00:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

All specifications and code of the services are on github.com but we are going to
redo it again by following the steps in the tutorial. Let's first create a workspace. 
I have created a directory named networknt under user home directory.

Checkout related projects.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-oauth2.git
git clone https://github.com/networknt/light-docker.git
git clone https://github.com/networknt/light-config-test.git

```

As we are going to regenerate API AA, AB, AC and AD, let's rename ms_aggregate folder from
light-example-4j.

```
cd ~/networknt/light-example-4j/rest/openapi
mv ms_aggregate ms_aggregate.bak
cd ~/networknt
```

In the next step, we are going to review the [specifications][] for these four services. 

[specifications]: /tutorial/rest/openapi/ms-aggregate/specification/
