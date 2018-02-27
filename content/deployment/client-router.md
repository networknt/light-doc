---
title: "Client module vs light-router"
date: 2018-02-26T20:20:02-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "deployment"
    weight: 50
weight: 50
aliases: []
toc: false
draft: false
---

High throughput and low latency are [principles of light platform][] and when it comes to
service to service communication, we recommend to use client side service discovery and load
balancing. 

light-4j has a [client module][] called Http2Client that can be embedded into any Java application
to handle the communication to services built in light platform. It handles HTTP 2.0 connection
and OAuth 2.0 provider communication as well as traceability in the service to service call tree.

However, the client module is not suitable for all scenarios as it requires that consumer application
must be built in Java 8 and above. This is not an issue for service to service communication
if both are built in light platform. But what about other consumers like applications built with
lower Java version, or built with other languages. 

For other languages, we are trying to provide client modules based on priority. However, if you
want to consume services now, you can leverage [light-router][] which is a client module at network
level.

You have two options to deploy light-router: 


* Deploy on the client host

In this setup, the light-router instance is private for the client and it can handle the token
renewal just like you have an embedded client module. Also, if there is no network hop, the
performance would be better than the next option. In this scenario, the consumer owns the router
instance. 

* Deploy in another network host

In this setup, the service provider owns the router instance. There is an extra network hop between
client and router so latency will be increased. Also, as the router instance is not owned by the
consumer, you cannot deploy client_id and client_secret on the router to handle the JWT token renewal
from OAuth 2.0 provider. 


[principles of light platform]: https://doc.networknt.com/about/principles/
[client module]: /concern/client/
[light-router]: /service/router/
