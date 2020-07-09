---
title: "Light Portal"
date: 2018-01-14T21:14:46-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 20
weight: 5	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

Light-portal is a service for consumer and provider runtime management and marketplace to bridge between consumers and providers. When you build microservices in a large scale on top of light-4j and related frameworks, you need a portal service to manage the entire life-cycle of your services and to provide a marketplace for bridging API consumers and API providers. We have explored the open-source projects on github.com and could not find any existing product that meets our requirement, so we decide to build it on top of the light-4j framework.   

In the long run, we will be deploying the infrastructure services to the cloud along with light-portal to serve small and medium-sized organizations at https://lightapi.net. All services in portal support multi-tenancy, so all users belong to one or more organizations on the portal. 

We will also set up a dev/test environment in parallel with https://lightapi.net called https://dev.lightapi.net for developers and testers. All-new UI components and services will be deployed here to test and then move to the production site. Also, this site is free for all users to access and test the functionality of their services. 

For some organizations that want to have their domain name and customized portal UI, the light-router who serves the portal UI supports [virtual host][]. In fact, https://lightapi.net is just one of the virtual hosts along with https://demo.taiji.io, https://faucet.taiji.io, and https://taiji.io on the same light-router instance. You can check the IP address for the above sites. With a virtual host, it would be very easy to have your portal site like https://portal.example.com with your company's customized UI style. 

We will also package the services and UI as an enterprise edition for large organizations to install on their private or public cloud. We will charge a yearly subscription fee to allow users to use the software and provide commercial support for customers. 

At the moment, all services are located in the light-portal and open-sourced to paid customers. And UI is located in portal-view open-sourced to the public on GitHub.

As we are aiming light-portal as a cloud service, it needs to scale for the Internet volume. Please visit [technology stack][] for the non-functional requirement and technology stack for both back-end services and front-end UI. 

The following are major components for light-portal:

* [API Certification](/service/portal/api-certification/)
* [User Management](/service/portal/user-management/)


[technology stack]: /service/portal/technology-stack/
[virtual host]: /tutorial/router/light-portal/
