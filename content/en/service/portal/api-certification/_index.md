---
title: "API Certification"
date: 2019-03-07T12:09:07-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This service is part of light-portal, and it is used to check if the production deployment meets the security requirements.

The following things will be checked with the server /info endpoint:

If TLS https is enabled and key pair are replaced
If HTTP port is disabled
If JWT token verification and scope verification are enabled
If metrics are enabled
If the right version of JDK/JRE is used to start the server
If the default JWT key is replaced

