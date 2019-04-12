---
title: "Hybrid - Modularized Monolithic"
date: 2017-12-20T21:21:09-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 05
weight: 05	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

For most organizations, switching from monolithic architecture to microservices is really hard as the initial infrastructure investment is very high and almost all the design patterns in Java EE are anti-patterns in microservices. It might be easier to start with modularized monolithic and then split each module to separate instances as microservices once volume picks up. The light-hybrid-4j is designed for it. It takes advantages of both monolithic and microservices with simple RPC style of interaction between services.

The light-hybrid-4j is a framework built on top of the light-4j for modularized monolithic and serverless architecture. The idea is that you first create a server with all the third-party dependencies included in the instance. Then you can build multiple services with the same dependencies or a subset of dependencies for development, unit tests, and integration tests. Once your services are completed, they will be built into small jar files without any dependency but only the business handlers. You drop these jars into a service folder on the host the server is running, start the server and all services in the folder will be loaded, and the traffic will be routed to the right handler in the right service for the request. 

Similar to the light-graphql-4j, the JSON based light-hybrid-4j is a particular case of RESTful API with all the information defined in the body of the request instead of scatted in the path parameters, query parameters. It gives us a uniformed request body structure so that we can quickly and efficiently identify the handler for a particular request. As JSON is typed, we don't need a very complicated Swagger specification to describe the variable conversion between service and consumer. 

For each service, you need to define a specification file which contains JSON schema for the light-hybrid-4j to validate the incoming request. This schema can also be used by the [react-schema-form][] to generate the application in a React SPA. 

Light-hybrid-4j is the framework we used most for our internal services. It gives us the flexibility for deployment and saves us the production cost. For example, the light-portal contains numeric services, but we only split them into a command server and a query server for now. It saves us a lot compare with light-rest-4j as we have to deploy dozens of containers instead of only four(two instances of command and two instances of query) in the beginning. In the long run, we have the option to scale the query server to more instances or split it to multiple servers if some services are hit with high volumes. 

Light-hybrid-4j contains the following components.

### rpc-router

rpc-router is a middleware handler that implement light-4j [handler interface][] and it will be inject into the request/response chain. It is responsible for parsing the incoming request body and then route to the request to the right handler to handle it. The handler will be implemented in one of the service jar file included into the server instance. 


### rpc-security

rpc-security is responsible for JWT token verification from Authorization header.  

- [Getting Started](/getting-started/light-hybrid-4j/)
- [Merge Multiple Services](/tutorial/hybrid/merge-services/)
- [JSON Protocol](/style/light-hybrid-4j/json-rpc/)
  * [Schema Definition](/style/light-hybrid-4j/json-schema/)
  * [Code Generator](/tutorial/generator/)
  * [Start Server](/style/light-hybrid-4j/start-server/)
  * [Test Service](/style/light-hybrid-4j/test-service/)
  * [Tutorial](/tutorial/hybrid/)
	  + [Merge two services with one server](/tutorial/hybrid/merge-services/)
	  + [Hello World Hybrid Server and Services](/tutorial/hybrid/hello-world/)
	  + [Eventuate Todo-list Hybrid Implementation](/tutorial/hybrid/todo-list/)
	  + [Portal Hybrid Service initialization](/tutorial/hybrid/hybrid-service-initial/)
	  + [Portal Hybrid Query with Light Codegen Web](/tutorial/hybrid/codegen-web-portal/)
	  + [Portal Hybrid Query with API Certification](/tutorial/hybrid/certification-portal/)
	  + [Portal Hybrid Command with Host Menu](/tutorial/hybrid/host-menu-command-portal/)
	  + [Portal Hybrid Query with Host Menu](/tutorial/hybrid/host-menu-query-portal/)
	  + [Portal Hybrid Command with Schema Form](/tutorial/hybrid/schema-form-command-portal/)
	  + [Portal Hybrid Query with Schema Form](/tutorial/hybrid/schema-form-query-portal/)
	  + [Portal Hybrid Service with User Management](/tutorial/hybrid/user-management-portal/)
	  + [Taiji Blockchain Web Client Server SPA with handler.yml](/tutorial/hybrid/web-client-spa/)

- [Binary Protocol](/style/light-hybrid-4j/binary-rpc/)

[handler interface]: /concern/handler/
[react-schema-form]: https://github.com/networknt/react-schema-form
