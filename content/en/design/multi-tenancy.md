---
title: "Multi-Tenancy"
date: 2018-02-13T20:50:25-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "design"
    weight: 60
weight: 60
aliases: []
toc: false
draft: false
reviewed: true
---

Multi-tenancy is an architecture in which a single instance of a software application serves multiple customers. Each customer is called a tenant. Tenants may be given the ability to customize some parts of the application, such as the color of the user interface (UI) or business rules, but they cannot customize the application's code.

Multi-tenancy can be economical because software development and maintenance costs are shared. It can be contrasted with single-tenancy, an architecture in which each customer has their own software instance and may be given access to code. With a multi-tenancy architecture, the provider only has to make updates once. With a single-tenancy architecture, the provider has to touch multiple instances of the software in order to make updates.

In cloud computing, the meaning of multi-tenancy architecture has broadened because of new service models that take advantage of virtualization and remote access. A software-as-a-service (SaaS) provider, for example, can run one instance of its application on one instance of a database and provide web access to multiple customers. In such a scenario, each tenant's data is isolated and remains invisible to other tenants.

In a private cloud, the customers, who are also called tenants, can be different business divisions inside the same company. In a public cloud, the customers are often entirely different organizations. Most public cloud providers use the multi-tenancy model. It allows them to run one server instance, which is less expensive and makes it easier to deploy updates to a large number of customers.

