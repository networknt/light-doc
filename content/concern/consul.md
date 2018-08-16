---
title: "Consul Client"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 30
weight: 30	#rem
aliases: []
toc: false
draft: false
reviewed: true
---


It is a consul registry implementation that uses HashiCorp Consul as registry and discovery server. It implements both registry and discovery in the same module for Consul communication. The server module is responsible for registering itself during startup and deregistering during shut down. 

In some cases, if the server is crashed, there is no chance for the server to invoke the deregister endpoint on the Consul agent. A health check must be configured to sure that Consul service catalog reflects the current status of the service instances. 

The framework provides three options to update health status for a particular service instance. For more details on which option to choose for your service, please see the comment in the consul.yml below. 

* TCP

The consul agent will check if the port number is open on a particular IP periodically. 

* HTTP

The consul agent will send an HTTP request to the /health endpoint on the service periodically. 

* TTL

The consul agent set up a TTL and expecting service to send heartbeat to the Consul check API before the TTL periodically. 

### Interface

Here is the interface of Consul client. 

```
public interface ConsulClient {

	/**
	 * Set specific serviceId status as pass
	 *
	 * @param serviceId service id
	 */
	void checkPass(String serviceId);

	/**
	 * Set specific serviceId status as fail
	 *
	 * @param serviceId service id
	 */
	void checkFail(String serviceId);

	/**
	 * register a consul service
	 *
	 * @param service service object
	 */
	void registerService(ConsulService service);

	/**
	 * unregister a consul service
	 *
	 * @param serviceid service id
	 */
	void unregisterService(String serviceid);

	/**
	 * get latest service list
	 *
	 * @param serviceName service name
	 * @param lastConsulIndex last consul index
	 * @return T
	 */
	ConsulResponse<List<ConsulService>> lookupHealthService(
			String serviceName, long lastConsulIndex);

	String lookupCommand(String group);

}

```

### Implementation

The implementation is simple restful API invocation with Http2Client in the light-4j client module. 

### Configuration

To map the ConsulClientImpl to the interface ConsulClient, there is an entry in the service.yml config file. The following is an example. 

```
singletons:
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry

```

Here is an example of service.yml in test folder for Consul module to define that Mocked Consul client is used for ConsulClient interface.

```
singletons:
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.MockConsulClient:
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
```

There is also a consul.yml to control the behaviour of the consul client. It is important to understand the difference between three options for the health check in this config file. 

Here is an example of consul.yml

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: true
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
```

