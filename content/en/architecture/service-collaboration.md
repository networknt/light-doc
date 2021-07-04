---
title: "Service Collaboration"
date: 2018-02-05T08:42:44-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

The recent trend in application architectures is to transition from monolithic applications to a 
microservices model. Software development teams are trying to develop a unified architecture that 
will work for most of the use cases in real world applications: online transactional, batch, IoT 
and big data processing pipelines.

This transition without a good service interaction model will most likely result in chaos and a 
service landscape that's hard to govern and maintain.The teams would have hundreds of microservices 
communicating with each other without any governance to allow only the authorized microservices to 
call the other microservices.

The following article is to discuss the pros and cons of service orchestration vs. service choreography 
and best practices on how to implement business process services that require calling multiple 
microservices. 

Let’s assume you want to build a simple order system covering the whole order fulfillment process with 
the following three microservices involved:

* Payment
* Inventory
* Shipping


The overall order fulfillment definitely requires these three services to collaborate. Currently there is 
a big fight going on as to which collaboration style to use — orchestration or choreography (for an 
introduction on the basic styles see [Orchestration or Choreography][]). 



[Orchestration or Choreography]: http://plexiti.com/en/blog/2017/03/microservices-orchestration-or-choreography/