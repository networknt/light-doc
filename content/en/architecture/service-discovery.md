---
title: "Service Discovery"
date: 2018-02-14T09:12:55-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 100
weight: 100
aliases: []
toc: false
draft: false
reviewed: true
---

When introducing distributed microservices architecture to an organization, we need to decide if we want to leverage service registry and service discovery in the cloud. If the answer is yes, we need to choose between client-side discovery vs server-side discovery. 

Regardless of server-side or client-side discovery, the registration is the first step. It will connect to the control pane during the server startup and send the protocol, I.P. address, port number, and other health check parameters.

There are three levels of usage for the registered information from the registry in a cloud environment. 

Query : 

The registry is used for queries and reports only. For example,  check the version of the framework to detect any need to upgrade due to security vulnerabilities.

Invocation: 

The registry contains all the access info to establish connections to the individual service instance. Regardless of server-side discovery or client-side discovery, we can use the registry to discover the available service instance to establish the connection to invoke API endpoints. 

Control:

With the ability to connect to any individual service instance, we can use the registry to control the service behaviours through admin API endpoints exposed by the service. For example, change the logging level for a particular package to diagnose a production issue or trigger chaos monkey attacks on production. 


The following are the features of service discovery in a distributed platform. 

### Auto Registry

It will try to get the podIP from the downward API passed to the server through the environment variable during the server startup. The server knows the port number, either dynamic allocated or statically bound. From the configuration, the server knows the protocol that is HTTP or HTTPS. With all the information above, it will establish a connection to the registry and send the connection info and health check info. 

If the registry is not available, the server should still start based on a configuration `startOnRegistryFailure: true` in the server.yml file if the registry is used for query only. For other advanced usages, we need to ensure high availability for the registry for services to register.

### Query APIs

The control pane exposes some query APIs to return all registered APIs or based on certain conditions. 

### Marketplace Integration

With the registered info from the control pane, the marketplace can access the control pane API to get all the available instances and render the swagger-ui to allow users to access business endpoint from the marketplace.

### Security Patch

When you have hundreds of services deployed, to ensure that all services are patched for the latest security vulnerabilities are very crucial. Right after the service registration to the control pane, the control pane will invoke the service /server/info endpoint to get the server info. It contains all the moduel configuration for the instance and the framework version with JDK/OS info. The information will be parsed to alert the platform team to patch the standalone API or service mesh sidecar. 


### High Availability

When the control pane is used for additional features, more than just query the available APIs in the cloud, we need to ensure the high availability for the control pane service. For a particular API instance, it can only register itself to one instance of the control pane during the startup. It requires the registry info to be replicated between different control pane instances. To ensure real-time replication, we use the Kafka streams processing on each instance. Once a new instance is registered on a control pane instance, it will push it to a Kafka topic. All instances will subscribe the topic with Kafka stream to update its in-memory cache. With this approach, we can guarantee the sync between instances, even after an instance is restarted. 

### Security Model

The control pane has a UI protected with OAuth 2.0 authorization code flow. The OAuth 2.0 provider should integrate with Active Directory so that the group from AD can limit access to certain APIs and endpoints. 

As each user has different groups from the Active Directory, the JWT token will have different claims. The control pane UI will render only the APIs visible to the correct claims in the JWT token to give the user access to the APIs he/she owns. 

Admin users who belong to the API platform support team will have a particular group from the AD that maps to the custom claim in the JWT token to allow them to see all the APIs available on the control pane. 

When accessing the admin endpoints for each API instance, the scopes in the token will be different for the above groups. For admin users, the scope will contain `admin` to allow access to all APIs. For API owners, the scope will contain `xxxxxx.admin`.  The `xxxxxx` is the API Id uniquely allocated for each API. 

The platform injects the admin scope for each API, so the API can verify the scope against to token to determine if access is granted or not. 

### Operation Model

There are multiple groups of people who will access the control pane with different permissions. One group is the platform support team that can access all the APIs, and other groups will be the individual API owners who will only access their APIs. 

As the security model describued, 

### Admin Endpoints

All the services built on top of light-4j frameworks, including the light-mesh sidecars, can inject the following admin endpoints to manage the services through the Control Pane. All the admin endpoints are protected with OAuth2 Client Credentials token with two types of scopes described in the above Security Model to allow the Platform Support team to access all APIs and API owners to access the APIs they own.

* Health Check

Light-4j provides a generic implementation of the health check handler. It can be exposed in the handler.yml configuration with different options. 

/health/{serviceId} used by Control Pane to uniquely identify a service container instance in a Kubernetes cluster with a client credentials token. 

/health/liveness and /health/readiness for Kubernetes probe without security protection.  

When the sidecar is used, there would be three endpoints exposed on the sidecar. 

/heath for Kubernetes probe for the sidecar container without security

/health/liveness/{serviceId} and /health/readiness/{serviceId} will pass through to the backend API /health/liveness and /health/readiness. The endpoint on the backend API can be used for the Kubernetes probe without security. The security is enabled on the sidecar with an admin token for Control Pane to access. 


* Server Info

The server info endpoint outputs all the environmental variables, configurations and metadata about the API (OS, JDK, Frameworks). After an API (backend API or sidecar on behave of backend API) register itself to the Control Pane, the control pane will invoke the `/server/info` endpoint to get the detail about the server. The information is aggregated and can be queried and reported from the Control Pane UI. Also, the information can be pass through a certification process to identify any configuration issues or security vulnerabilities. The API Platform team can decide to upgrade API or sidecar based on the certification result. 

When the sidecar is used,  the `/server/info` needs to be implemented on both. The endpoint on the sidecar will combine both backend server info and sidecar server info together. 

* Chaos Monkey

By default, we can wire in the Chaos Monkey attack handlers to the request/response chain and disable them. With Chaos Monkey endpoints, we can enable specific attacks and control the parameters for the attacks. After the tests, we can disable the Chaos Monkey again. 

* Logger Config

Two endpoints are injected to query logging levels for a particular package and update the logging level for any specific Java package. It is a critical feature for the API owner to diagnose the production issues without enabling debug/trace all the time. 

Not a lot of logging output from the sidecar in a normal situation; however, users can decide to pass through the request to the backend API to manipulate the logging levels.

* Sidecar Admin

There are specific admin endpoints available for some sidecars and can be accessed from the Control Pane. For example, the Kafka Sidecar has many endpoints to change the offset, manually sign partitions and replay dead letters. 


### Service Discovery

The ultimate goad of service registry is for service discovery. In a distributed microservices application, there would be service to service invocation everywhere. Instead of calling downstream services with Ingress or service in a Kubernetes cluster, we can directly invoke downstream API  with IP and port with the lookup from serviceId from Control Pane. 

The service discovery can be made on the sidecar instead of each backend API implementation. When invoking a downstream API in the backend API, it will just invoke localhost with the sidecar port. The sidecar will translate the path to a serviceId from the configuration, look up the service, load balance between multiple hosts, establish a connection, cache the connection, subscribe to any changes for the downstream API on the Control Pane. 

As described above, the connection is cached and reused to reduce the latency to a minimum. However, we don't want to use the same connection forever. To ensure that the load is balanced between different nodes, we need to drop the connection after a certain number of requests (1 million) or a specific time ( 1 hour) from the client.yml configuration file. 

After a client subscribes to a downstream API, the Control Pane will notify the client of any changes for the downstream API through WebSocket. It ensures every client has the latest snapshot of downstream APIs in the cache. When creating a new connection, there is no need to have an active lookup but just use the local cache to establish a new connection. 


