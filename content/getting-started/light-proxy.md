---
title: "Light Proxy"
date: 2017-11-05T12:31:31-05:00
description: ""
categories: [getting started]
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

For an organization that established infrastructure to support services built on
top of light platform, chances are they have some existing services built with
other platforms or even other languages. In order to leverage the security, metrics,
logging, auditing, client side service discovery etc. for these existing services, it is
a good idea to put [light-proxy][] in front of these services to provide cross-cutting
concerns for them. Although the light-proxy adds an additional network hop, however, it
might be faster given it support HTTP 2.0 by default. 

A proxy server is a go‑between or intermediary server that forwards requests for content from 
multiple clients to different servers across the Internet. A reverse proxy server is a type of 
proxy server that typically sits behind the firewall in a private network and directs client 
requests to the appropriate backend server. A reverse proxy provides an additional level of 
abstraction and control to ensure the smooth flow of network traffic between clients and servers.

## Common uses for a light proxy server include:

* Support Swagger 2.0, OpenAPI 3.0, GraphQL and RPC

With the same code base, it support different style of API interactions by changing configuration
only. In light*4j, different frameworks share a same set of common middleware handlers and have
their own specific handlers. All of them can be wired in as plugins in service.yml config file
to form different type of services. By wiring in framework specific handlers, light-proxy can be
used with RESTful Swagger 2.0 and OpenAPI 3.0, GraphQL or RPC backend.  

* Load balancing 

A reverse proxy server can act as a “traffic cop,” sitting in front of your backend servers and 
distributing client requests across a group of servers in a manner that maximizes speed and capacity 
utilization while ensuring no one server is overloaded, which can degrade performance. If a server 
goes down, the load balancer redirects traffic to the remaining online servers.

* Web acceleration

Reverse proxies can compress inbound and outbound data which speeds up the flow of traffic between 
clients and servers. They can also perform additional tasks such as SSL encryption to take load off 
of your web servers, thereby boosting their performance. When using HTTP 2.0 protocol, data over 
Internet between client and proxy server is in binary and headers are compressed. This saves the 
bandwidth and provide better performance even though the backend service is still in HTTP 1.1 protocol.

* Security 

By intercepting requests headed for your backend servers, a reverse proxy server protects their 
identities and acts as an additional defense against security attacks. It also ensures that multiple 
servers can be accessed from a single record locator or URL regardless of the structure of your local 
area network. In API world, this means the proxy server is responsible for authorization on the client
and scope. 

* Metrics collection

Light-4j metrics middleware handler can be placed into the request/response chain to collect the successful
requests, failed requests and response time from both client_id and service_id perspective. From the
Grafana dashboard, service owner can see how many clients are accessing the service and response code
distribution, volume as well as response time. For client owner, it can see how many APIs the client is 
calling and the corresponding response code distribution, volume and response time. 

* Centralized Logging and auditing

The auditing handler can be enabled in the request/response chain to intercept the request and response
and logged to audit.log or database based on the audit handler implementation. This gives insight how
the existing API is access and provide auditing information in the same format with other services built
on top of light-*-4j frameworks. 

* Request Validation

All different style of API frameworks will have schema validation again the request and it is done by 
the validation handler in each framework. For example, if existing backend service is RESTful API, then
you can create the swagger specification and enable swagger handler and validator handler to validate
the request before reaching the backend service. 

* DDos mitigation

You can enable rate limiting handler on the proxy server to ensure that high volume of requests will
be throttled. This normally only be enabled if your service is exposed to the Internet through the
proxy server. 

* Static IP service

As most light services will be Dockerized and deployed on the cloud. There is no static IP addresses
and port number available. For Mobile native applications or Single Page application, it would be easier
to address a reverse proxy server which has a static IP address and provide service discovery to forward
the request to the right IP and port number. (working in progress)

* Serve static content

Like the static IP servie, the proxy server can act as a webserver as well to provide static content
to serve single page application and assocated contents. (working in progress) 

In summary, the light-proxy provides the features of generic reverse proxy servers like Nginx or
HAProxy and at the same time, provide better performance and a lot of cross-cutting concerns other
proxies won't have. 

To learn more on how to configure each feature in the configuration files. Please refer to [Configuration][]
and [Tutorial][]. To find out all the deployment options and choose the right one, please refer to [Artifact][] 



[light-proxy]: /service/proxy/
[Configuration]: /service/proxy/configuration/
[Tutorial]: /tutorial/proxy/
[Artifact]: /service/proxy/artifact/
