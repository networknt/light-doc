---
title: "API Gateway"
date: 2017-11-06T11:14:26-05:00
description: "API Gateway is embedded in light-4j framework"
categories: []
keywords: [architecture]
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

When your organization is thinking about breaking up the big monolithic application to adopt microservices architecture, chances are there are some vendors coming to sell their gateway solutions. Why they want to sell you gateways and do you really need a gateway?

The reason they are eagerly selling you a gateway is that they have to monetize their product as soon as possible before it is obsolete. The solutions they provided are not for true microservices as there is no standalone gateway in the picture of the real microservices architecture. Their solution is coming from web services(SOA) design and all services behind the gateway are flattened.

The purpose of the gateway is to address cross-cutting concerns in Web services as these are built on top of JavaEE platform with no capability to address these concerns inside the application server. The foundation of JavaEE is Servlet API which treats request and response immutable and it doesn't provide a way to intercept request and response easily in the servlet filter. The servlet API was designed around the year 2000, and it cannot be changed due to backward compatibility guarantee. This leaves only one option to address cross-cutting concerns on the network level. Normally JavaEE apps are designed as multiple layers and putting an extra layer as a gateway is very natural. 

Here is a picture of their typical solution in the beginning.

![ms_oauth2_gateway](/images/ms_oauth2_gateway.png)

After a while, they realized that for every request from the client, there are two calls from client and service to OAuth2 server and remote calls are too heavy.

Then the solution for gateway vendors is to move OAuth 2.0 server inside the gateway so that there are no remote calls between the gateway and OAuth 2.0 server for security verification. Here is an updated gateway.

![ms_oauth2_in_gateway](/images/ms_oauth2_in_gateway.png)

With increasing volume, the monolithic gateway becomes a bottleneck, and the only solution is horizontal scaling. That means you have a cluster of gateway instances and the gateway becomes a single point of failure. If any component fails in the gateway, all your APIs are not accessible even they are all up and running. It is also very hard to scale as it is a big monolithic application with a lot of components built in and uses a lot of CPUs and memory. 

Another issue with the monolithic gateway products is that most of them are built on top of Java EE platform. The blocking nature of servlet containers will have stuck thread problems if backend services are slow. For example, if there are several single page applications, talk to several APIs behind a gateway. Suddenly one service has an issue and the response time drops from 20 milliseconds to 20 seconds. If there are several impatient users submitting requests repeatedly, all threads on the gateway can be stuck and the gateway cannot accept any new requests anymore. It not only blocks the application with performance issues but also blocks all other clients and services although these applications are perfectly healthy. This is the main reason we are not using a monolithic API gateway as a proxy but can only use it as an OAuth 2.0 provider.

The gateway vendor might argue that the majority of the organizations won’t have a big volume so it is OK to let all external requests pass through the gateway. It is true that most gateways support hundreds or even thousands requests per second. And they claim that 99% of organizations won’t have even 200 requests per second. The reason there was no huge volume in the past is due to the server side rendering application. These applications have a form with dozens of fields and you need to fill all of them and click the submit button to send the request to the server, then the validation is done on the server and errors will return to the browser. The UI is designed this way because server latency is very high, at around 1 to 2 seconds. There is no way we can do real-time validations. Once we move to APIs or microservices, our server response time reduces from several seconds to several milliseconds. With single page applications built in Javascript frameworks like React/Angular, we can validate the user input when the cursor moves away from the field. We can validate the user input while typing like a Google search. With APIs and SPAs, the number of requests from browsers to the internal services increases hundred or thousands times more than before. How could a single gateway handle multiple million requests per second to match a microservice throughput? How about a big organization has thousands of microservices?

Using a monolithic gateway with high throughput and low latency microservices is just like building a ten-lane highway but with only one entry point which allows one car per minute. Or you have a vehicle which can travel 100km/hour but must be pulled by a horse.

When you look inside the APIs protected by the gateway, you can see these APIs are implemented in JavaEE containers like WebLogic/WebSphere/JBoss/SpringBoot etc. and they don't call each other. They are simply monolithic JavaEE application packaged in ear, war or jar and exposed RESTful APIs. These APIs are normally deployed in Data Centers and recently moved to the cloud. They are not real microservices at all. 

Some smart developers attempted to break these big applications into smaller pieces and move in the direction of microservices, but gateways became a problem. Let's take a look at how API to API call looks like with a gateway in the following diagram.

![ms_gateway_api_to_api](/images/ms_gateway_api_to_api.png)

As you can see, when API A calls API B, although both of them are behind the gateway, the request has to go in front of the gateway to properly authenticate/authorize the request. Clearly, the centralized gateway design is against the decentralized principle of microservices architecture.

In our framework, the solution is to move all the cross-cutting concerns to the API framework and APIs are built on top of the framework. In other words, a distributed gateway. Here is a diagram to show you client calls API A and API A calls API B/API C and API B calls API D. 

![ms_distributed_gateway](/images/ms_distributed_gateway.png)


In this architecture, every API instance contains functions from the framework and act as a mini gateway embedded. Along with container orchestration tools like Kubernetes or Docker Swarm, the traditional gateway is replaced. As there are no remote calls between API to a centralized gateway, all the cross-cutting concerns are addressed in the same request/response chain. This gives you the best performance for your APIs. Here is a [tutorial][] which implements the above diagram and source code for the four APIs can be found in GitHub [light-example-4j repo][].

Our framework is built on top of Undertow HTTP core server which is very light and serves 1.4 million “Hello World!” requests on my desktop with an average response time of 2ms. It is 44 times faster than the most popular RESTful container Spring Boot with embedded Tomcat.

The performance test code can be found in [microservices framework benchmark][].

In the above diagram, OAuth2 server is an independent entity and you might ask if it is a bottleneck. I have written another security document to address it with distributed JWT token verification and client credentials token caching and renewal. Basically, the Client module in the framework caches the client credentials token until it is about to expire then renews it in the background.

Also, our own OAuth2 server built on top of light-rest-4j and light-hybrid-4j frameworks, which are very fast and can handle 60K user logins to get authorization codes per second. It can serve about 700 access tokens in a second. It is also open sourced and can be found at [light-oauth2 repo][].

[tutorial]: /tutorial/rest/swagger/ms-chain/
[light-example-4j repo]: https://github.com/networknt/light-example-4j
[microservices framework benchmark]: https://github.com/networknt/microservices-framework-benchmark
[security document]: /architecture/security/
[light-oauth2 repo]: https://github.com/networknt/light-oauth2
