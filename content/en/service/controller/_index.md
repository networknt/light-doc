---
title: "Light Controller"
date: 2020-11-13T19:55:55-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 71
weight: 71
slug: ""
toc: false
draft: false
reviewed: true
---

When deploying large-scale microservices to the cloud environment in a zero-trust network, the traditional pull from each service to check if it is healthy is not feasible. Instead, we need each service instance to register the IP and Port to the Portal Controller service through API calls during the server startup and de-register during the server shutdown. It requires the API framework to have a component that is responsible for registering and de-registering. We can also leverage the Portal Controller for the service discovery as the Controller contains all the access information for live nodes. With an API exposed from the Controller, API consumers can query the Controller to get a list of instances it is about to invoke. In the service to service call scenario, a service can register itself and, at the same time, query the Controller to discover downstream APIs. A service can be called by multiple other services or clients in a complex call stack, and it might call several other services to fulfill its requests. The Portal Controller can support all of the above and beyond.

For a service that wants to register to the controller during the server startup, you need a config file portal-registry.yml to define the controller's URL. You can set up the health check interval in the same file, how long the service should be removed from the controller if the health check fails, and what kind of health check is selected.

The controller supports HTTP and TTL health checks. The service needs to choose between them.

HTTP check is initiated from the controller to call each service's /health/{serviceId} endpoint, and it is recommended. TTL is for services running in a separated network, and the controller cannot access the health check endpoint. 

In light-4j, there is a module called [portal-registry](/concern/portal-registry/) that is responsible for the communication to the controller. You can find all the details on how to set up the module in your API in the cross-cutting concerns section.

The following is a sequence diagram that illustrates the service startup and shutdown activities. 

![portal-registry](/images/portal-registry-service.png)

In the above picture, we only draw one service as an example. In reality, there would be hundreds or thousands of services registering and deregistering themselves on the portal controller.

Because all live instances for each service are all registered to the portal controller, the controller has a snapshot of all healthy instances at any time. The information is very valuable for global service discovery in a cloud environment. For each client to invoke a downstream service, it can go to the controller to look up healthy instances based on the serviceId and an optional environment tag. This list can be cached for future usage at the client-side so that the client doesn’t need to call the controller for each downstream call. If the client uses Http2Client provided by the light-4j, the connection to the downstream API can be cached so that the same connection can be reused until a specific time (8 hours) or a certain number of requests (1 million) is reached. 

Service owners might scale their services up and down in a dynamic cloud environment to utilize the resources efficiently. It means the number of instances per service, the IP, and the port combination are all changing. All changes are captured in the controller and we need to find a way to notify the clients that their downstream service is changed.

There are two options for the notifications: Long poll (blocking query) and WebSocket. The Consul registry uses a blocking query so that each subscribed downstream service needs a thread to pull the controller constantly. This approach wastes a lot of CPU threads if there are multiple downstream APIs. Portal Registry uses WebSocket so that all the updates for the subscribers can share the same channel with only one callback to update the internal cache on the client.

Here is a picture of the client discovery and synchronization with the controller. 

![portal-discovery](/images/portal-discovery-client.png)


Please note that there is only one client in the picture to make the diagram simple and easy to understand. In a real environment, there would be hundreds or even thousands of clients in the picture. A service itself can be a client that calls several services, and several clients can call it. 

Although the light-controller is not an open-source project, it is free to use in a docker container. We also provide a docker-comopse-controller.yml in the light-docker project with all the configuration files externalized to the controller folder for customization. 

- [Getting Started](/service/controller/getting-started/)
- [High Availability](/service/controller/high-availability/)
- [Two Mode](/service/controller/two-mode/)
- [Service Registration](/service/controller/service-registration/)
- [Health Check](/service/controller/health-check/)

