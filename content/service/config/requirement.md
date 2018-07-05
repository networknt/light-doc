---
title: "Requirement"
date: 2018-07-04T14:47:07-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When managing the configurations, the following things need to be considered as part of the requirement. 

* Config server instance per environment

For each environement, there should be an instance of config server with config templates and key/value pairs for the environment. The authentication and authorization would be different so that developers can access DEV config server. Operators can access UAT config server and Production operators can access PROD config server. 

* Multiple config template files per service.

Each service has numeric plugins and each plugin has its configuration file to control if it is enabled and the runtime behavior. That means for each service, there would be a list of configuration files per environment like DIT, SIT, UAT and PROD. To ease the burden to manage different config files per environment, we would create config file templates instead and replace some of the values per environment from an environmental specific config server. 


* Every environment has some values different.
* Some sensitive values need to be encrypted.
* Some of the config files need to be merged from several organizations.
* Some of the config files need to be overwritten from several organizations.
* When config server is used, there should b minimum internal config files to ensure the connectivity to the config server
