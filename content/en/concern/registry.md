---
title: "Registry Discovery"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This module contains all the interfaces needed in registry and discovery. It also implements a direct registry, which you can hard-code services into the service.yml or direct-registry.yml to simulate Consul or Portal Registry during development. Although this is for local development, many users are still using it during production when they have allocated services to the exact IP and port on virtual machines.

Currently, [Consul](/concern/consul/) and [portal-registry](/concern/portal-registry/) are supported for external service registry and discovery. When Consul Server or Light Portal is unavailable, you can use the [Direct Registry](/concern/direct-registry/) as a temporary solution. 


