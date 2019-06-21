---
title: "External Config"
date: 2018-08-10T09:05:08-04:00
description: "Externalize the configuration files for services deployed to Kubernetes cluster and discovered with a production ready consul cluster."
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous steps, all config files are packaged internal in the resource/config folder. This is very convenient for testing; however, it is not for the production. Light-4j supports externalized configuration with ConfigMap and Secret on the Kubernetes platform. In this tutorial, we are going to externalize some config files related to service discovery. 


In the next step, we are going to enable the [security][] with service discovery as a lot of users are asking how to integrate the security in a distributed system. 


[security]: /tutorial/common/discovery/security/
