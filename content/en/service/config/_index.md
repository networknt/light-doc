---
title: "Config Server"
date: 2018-02-05T11:32:16-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 60
weight: 60	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

Services built on Light are composed with many plugins as part of the embedded gateway, and each plugin (or middleware handler) has its configuration file and runtime behavior to control if they are enabled. Developers usually don't need to worry about these config files as these can be added later on by the operation team.

Unlike other application frameworks, we are not using a single config file but multiple config files as each service decides which plugins should be utilized and how they are utilized. 

When managing the configurations, the following things need to be considered:

* Multiple config template files per service.
* Every environment has some different values.
* Some sensitive values need to be encrypted.
* Some of the config files need to be merged from several organizations.
* Some of the config files need to be overwritten from several organizations.
* When the config server is used, there should b minimum internal config files to ensure the connectivity to the config server


- [Requirement](/service/config/requirement/)
- [Design](/service/config/design/)
- [Process](/service/config/process/)
- [API Doc](/service/config/api/)
- [service verify](/service/config/verify/)


