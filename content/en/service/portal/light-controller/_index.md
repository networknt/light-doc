---
title: "Light Controller"
date: 2020-11-13T19:55:55-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When deploying large-scale microservices to the cloud environment, the traditional pull from the Portal Controller to each service is not feasible. Instead, we need each service instance to push the IP and Port to the Portal Controller service through API calls during the server startup and shutdown. It requires the API framework to have a component that is responsible for registering and de-registering. We can also leverage the Portal Controller for the service discovery as the Controller contains all the access information for live nodes. With an API exposed from the Controller, API consumer can query the Controller to get a list of instances it is about to invoke. In service to service call scenario, a service can register itself and, at the same time, query the Controller to discover downstream APIs. A service can be called by multiple other services or clients in a complex call stack, and it might call several other services to fulfill its requests. The Portal Controller can support all of the above and beyond.

