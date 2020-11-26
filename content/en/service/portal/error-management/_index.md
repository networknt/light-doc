---
title: "Error Management"
date: 2020-11-24T20:03:38-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Light-4j has a module named status that is used to manage the error message returned from the API. It loads the status object from the status.yml file in the status module along with the app-status.yml defined by each service built on top of light-4j frameworks. 

Here is an example of status defined in the status.yml

```
ERR10000:
  statusCode: 401
  code: ERR10000
  message: INVALID_AUTH_TOKEN
  description: Incorrect signature or malformed token in authorization header

```

