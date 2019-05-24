---
title: "Development Flow"
date: 2017-11-06T17:18:18-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "development"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
---

There are three flows running in parallel but not started at the same time in API development

API Specification starts the first and it will be done by data architect and business
analyst.

API implementation starts when the first release of the specification is done by API 
developers.

Client implementations start almost the same time as API implementation team for mock API can
be generated from swagger specification immediately.

During the process, specification might be changed and the changes will be propagated
to API implementation team and Client implementation team(s).


![API Flow](/images/api_flow.png)