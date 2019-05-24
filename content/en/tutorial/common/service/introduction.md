---
title: "Introduction"
date: 2017-11-13T22:31:45-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The light-4j service module is a singleton service factory which is an IoC service
to allow developers and operators to switch implementations of a specific interface
in service.yml config file. 

You can have two implementations for one interface and two service.yml file in both
main and test folder to use one instance for release and another instance for test.

Or you can have a debug implementation in your service and enable it on production
when something is wrong. Or switch two different data sources on the fly.

For more detail on the module, please take a look [service module][].

[service module]: /concern/service/

