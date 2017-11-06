---
title: "Mask"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [Concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 75
weight: 75	#rem
aliases: []
toc: false
draft: false
---

In the entire life cycle of the exchange, there might a lot of logging statements
written to log files or other persistence storage. These logs will be used to
assist production issue identifying and resolving and a wide group of people might
have access to these logs. In order to reduce the risk of leak customer info,
sensitive info needs to masked before logging. For example, credit card number,
sin number etc.

# Configuration

Given different API will have different sensitive data, the mask is configurable
and can be applied at header, cookie, query parameters and body.

# Mask with String

# Mask with Regex

# Mask with JsonPath

# Mask with Map and List

