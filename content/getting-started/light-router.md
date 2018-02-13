---
title: "Light Router Server"
date: 2018-02-11T16:27:10-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 70
weight: 70
sections_weight: 70
aliases: []
toc: false
draft: false
---

While we have provided a client module in light-4j for service consumer to interact with service
provider, there are certain scenarios that it cannot be utilized. For example, the consumer is a
PEGA workflow application that cannot easily include client.jar or the consume is an service that
is implemented with Dotnet which cannot leverage the client.jar

We are in a process to provide client modules for other languages and even provide frameworks for
other languages; however, it needs time for these solutions to be production ready.

In order to resolve the current issue, we decide to provide a router for the clients that cannot
leverage client.jar in light-4j. This is how light-router is come to light. 

As this is a client router, it should be deployed on the same machine on the client and it can
provide service discovery and routing for all the services this particular client is accessing
to. This architecture also save one network level round trip as your client and light-router
communication is local. Another options is to put it in your data center with a fixed IP address
to discovery and route to the services dynamically deployed in the cloud. 

Compare with the client.jar direct connection, the performance with light-router will be really
bad and we will publish some benchmark data later. Also, the cost of deploy another service and
support it need to be counted as well.

You can find more details about [light-router][] in service section. 

Also, if you want to learn how to use it, there is a [tutorial][] to show you steps to get it
implemented. 



[light-router]: /service/router/
[tutorial]: /tutorial/router/


  


