---
title: "Killapp Assault"
date: 2021-01-28T22:22:46-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

With the following config file, one of the requests will trigger the handler and kill the application process. 

killapp-assault.yml

```
# Light Chaos Monkey Kill App Assault Handler Configuration.

# Enable the handler if set to true. Otherwise, the request will pass through.
enabled: false
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5

```


