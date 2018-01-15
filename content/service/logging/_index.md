---
title: "Centralized Logging"
date: 2017-11-18T16:29:09-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 30
weight: 30	#rem
aliases: []
toc: false
draft: false
---

Logging is very important for large scale enterprise applications as it is crucial for
support team to figure out what has happened if something is wrong with the application. 
Once we move from the monolithic architecture to microservices architecture, logging
becomes more complicated as we cannot allow support team to go to dozens or hundreds
VMs or Docker containers to check the log in order to diagnose issues. A centralized
logging is a must and some sort of tracing and correlating need to be built into the
log from each service.

 