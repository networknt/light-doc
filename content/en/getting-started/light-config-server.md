---
title: "Light Config Server"
date: 2017-11-05T12:33:29-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 80
weight: 80
sections_weight: 80
aliases: []
toc: false
draft: false
---

Each plugin in Light has a configuration file to control if the component is enabled and its behaviour during runtime. Most modules have a default configuration file in the module src/main/resources/config folder that  can be overridden by the application level configuration with the same file name in the same location. Also, the configuration files can be externalized to a file system folder specified by a system property. To help manage these config files in a hierarchical structure, we have created a config server for services to load their config during startup.
