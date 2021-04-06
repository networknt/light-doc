---
title: "Service Mesh Plus"
date: 2019-01-10T14:19:11-05:00
description: ""
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 11
weight: 1
aliases: []
toc: false
draft: false
reviewed: true
---

In recent years, service mesh has begun to become popular, and a lot of organizations are trying to figure out how to adopt it in their microservices architecture. If you want to review the concept of the service mesh, please visit the [service mesh introduction][]

### Service Mesh vs Service Mesh Plus

One of our customers called light-4j service mesh plus as it is the next level of the natural evolution of service mesh. To understand it, let's first take a look at one of the most popular implementations - Google Istio. 

![istio-service-mesh](/images/istio-service-mesh.png)

In order to extract the technical cross-cutting concerns from the application, it puts two proxies between two services for service to service communication. If service A calls service B which is implemented with OpenAPI 3.0 specification, it needs: 

* Service A sends a REST request from its container to a proxy in another container associated with service A. 
* The service A proxy picks up the request, address all cross-cutting concerns and then transform it into a GRPC request to a proxy container that associated with service B container. 
* Service B proxy receives the request, address all cross-cutting concerns and sends the request to the service B. 
* The response from service B needs to go through the same route back to service A 


The following is how light-4j handles the functionalities of service mesh in a chassis instead of network proxies. 

![light-4j-service-mesh-plus](/images/light-4j-service-mesh-plus.png)

What we did is move all the cross-cutting concerns on the proxy server into the chassis and put them into the request/response chain so that they are called within the same process. They are implemented as plugins, and we call them middleware handlers. An individual application can pick up out of over 100 technical middleware handlers to wire into the application. These middleware handlers are all controlled by configuration, and the developers who are working on the business logic handlers won’t even notice their existence. The other benefit of this architecture is that different customers can implement their own middleware handlers running within the business context for its industry. For example, one bank has implemented fine-grained authorization and their own business auditing service on the request chain and response filter on the response chain based on a rule engine to give different client data filtered in rows and columns.

### History

To really understand how we reach the current architecture, we need to review the history of web service and microservices. As we all know, microservices architecture is derived from SOA (Service-Oriented Architecture) which is a monolithic based web service. It went through the following stages in the last several years.

* Monolithic API Gateway and OAuth2 Provider

This is not real microservices as all web services are flattened behind the gateway. A client needs to go to the OAuth2 provider to get the access token and send the request to the gateway. The gateway receives the request with the token and sends it to the OAuth2 Provider to verify the token and then proxy the request to the right service.

* Monolithic API Gateway with embedded OAuth2 Provider

With all the round trips for the service invocation, the throughput and latency are not good. Naturally, the gateway vendors gradually developed an OAuth2 provider of their own and embedded it into the gateway so that the network trip between gateway and OAuth2 provider can be saved although the query to the OAuth2 component is still on.

* Distributed API Gateway

With more and more services being developed, people started to realize that sending all requests through a monolithic gateway is not a good idea, especially as most gateways were built on top of Java EE technical stack. It allocates one thread per request and the stuck threads issue will occur if there is one bad service that responds very slowly. It causes the entire gateway to be stuck.

The solution for gateway vendors is to build smaller gateways so that they can have gateway instances per client or per service, or a group of clients and services.

* Service Mesh

The distributed gateway works for flattened web services or APIs, but they are not real microservices. Most APIs are still built on top of Java EE stack and implemented as monolithic at this stage. Once the monolithic is broken down to real microservices, there is service to service communication behind the gateway. The traditional gateway cannot address the service to service communication as most of the gateways are deployed on the DMZ to accept requests from the Internet. We cannot route the internal requests to the DMZ to address cross-cutting concerns like security. Also, the volume of the requests grows exponentially when the number of services increases. The traditional Java EE based gateways cannot handle the load anymore.

With container technology maturing, some companies have started to implement service mesh which places each service or a group of services on the same host gateway. These gateways/proxies are implemented with non-blocking with high throughput, low latency, and smaller memory footprints.


* Service Mesh in the Same Container

The service mesh resolved the scale issue for the gateway; however, it has its inherent issues as there are so many network hops for service to service communication. The serialization/deserialization reduces the throughput and increases the latency. If one external request needs to invoke 10 services within the network, they need to pass 20 proxies, and the latency will add up. How about one request that involves 100s of services? How many more containers need to be deployed if you have thousands of services with 10 thousands of instances? How many more costs will you incur for these proxies? How do you manage/monitor these proxies?

For the containerized application, the container itself uses a lot of memory as it has a virtualized Linux operating system. The next step for these vendors would be to move the proxy inside the application container to save the extra container. In that case, the application and the proxy will be running in the same container and communicate through the TCP/IP socket. This saves the extra container and makes it easy to manage and scale. I am guessing that there will be one or two products available on the market in 2019.

* Service Mesh in the Same Chassis

The above approach only solves the cost issue but still cannot resolve the throughput and latency issue. Light moves the proxy inside the application process with plugins. This allows service to service communication based on HTTPS/2 on a cached connection most of the cases. The connection supports multiplexing which means that one connection can support thousands of concurrent requests. The connection will only be recreated if the target service is down or after a certain period time or a number of requests configurable.

For legacy clients and services, Light provides the service mesh proxies in the container as well to bring them into the light ecosystem. light-proxy is used for legacy services to address cross-cutting concerns. For example, it can convert the OAuth2 JWT token to basic authentication and register the legacy service to the service registry so that others can find it on the cloud. We also have light-router which is built for external clients or clients that are not implemented in Java. It is responsible for security and service discovery and other cross-cutting concerns for the client. As you can see, even though we are using service mesh, we only need light-router or light-proxy and seldom use both unless both the client and service are not on Light.

### Pros and Cons

##### Serivce Mesh Pros

The benefit of the service mesh like Istio is that it supports applications that are implemented in different languages and frameworks. As it is working on the network level, it doesn’t know how your application is implemented. If your company adopts heterogeneous microservices, then service mesh is the solution to go.

##### Light-4j Cons

To adopt a chassis based service mesh platform like light-4j, it is required that all services be implemented in light-platform so that all of them have an embedded proxy inside. For most organizations, I think it is a good idea to choose one platform to implement microservices as there are a lot of things to learn and the cost will be much higher if multiple languages and platforms interact together. At the moment, we have a lot of cross-cutting concerns that can integrate with existing gateway products, and we are developing the light-portal to provide control plane features for the service mesh components. We can soon provide a hybrid solution to our customers that have the light-4j ecosystem and service mesh integrated tightly.

We are also working on other languages like Javascript, Go, .Net and Rust to build microservices frameworks following the same architecture of light-4j. Once we have other languages supported, these services will behave the same, and they can interact with each other seamlessly without any help from another network component.


[service mesh introduction]: /architecture/service-mesh/
