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


It is a service registry implementation that uses HashiCorp Consul as registry and discovery server. It implements both registry and discovery in the same module for Consul communication. The server module is responsible for registering itself during startup and deregistering during shut down. 

In some cases, if the server is crashed, there is no chance for the server to invoke the deregister endpoint on the Consul agent. A health check must be configured to sure that Consul service catalog reflects the current status of the service instances. The health check also allows Consul to deregister service after a configurable period if the service instance is in the critical state. 

The framework provides three options to update health status for a particular service instance. For more details on which option to choose for your service, please see the comment in the consul.yml below. 

* TCP

The Consul agent will check if the port number is open on a particular IP address periodically. This is the simplest option for Consul health check; however, it is not reliable in a dynamic cloud environment with containers. It should only be used with VM deployment on fixed IP addresses and there is no chance two different services will be deployed to the same IP with the same port number. 

* HTTP

The Consul agent will send an HTTP request to the /health/{serviceId} endpoint on the service periodically. This is the recommended approach for services deployed to the Kubernetes cluster with host network and dynamic port enabled. When Consul calls the endpoint, the serviceId is the path parameter so it guarantees that it is the right service. HTTP health check also gives you more accurate info about the server. For example, the service instance might be still listening to the port but no request can be processed. Or the service depends on a database which is down. If your service has dependencies, you need to customize the default Health Check handler from the framework to give the more accurate status of the service. 

* TTL

The Consul agent set up a TTL and expects service to send heartbeat to the Consul check API before the TTL expires periodically. This approach put a lot of load on the Consul server/agent and should be only used when you service cannot be accessed from the Consul agent. For example, if your client and service are running on another subnet. It is recommended that you deploy Consul agent to the same host as your service so that the load can be reduced on the Consul server cluster.  

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

The implementation is simple restful API invocation with Http2Client in the light-4j client module. If TLS is enabled on the Consul cluster, HTTP/2 will be used automatically for efficiency. In a test environment with HTTP connection to Consul, the connection will be downgraded to HTTP/1.1 and it won't support high throughput due to the lack of multiplexing. Consul is implemented in Go and it doesn't support HTTP/2 if http is used. It is highly recommend installing Consul with TLS and HTTP/2 in the beginning. For installation guide, please refer to this [Consul Installation Tutorial][]

### Configuration

* service.yml

To map the ConsulClientImpl to the interface ConsulClient, there is an entry in the service.yml config file. The following is an example.

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: consul
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
```

Here is an example of service.yml in test folder for Consul module to define that Mocked Consul client is used for ConsulClient interface. This is used in the unit test only. 

```
singletons:
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.MockConsulClient:
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
```

* consul.yml

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
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: true
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
```

* server.yml

In the server.yml, we need to enable registry and dynamic port. Here is an example. 


```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  7001

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  7441

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled by default.
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.apia-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: true

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: true

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1
```

Please note that enableRegistry is true and dynamicPort is true. Also, there is minPort and maxPort to define a range for port allocation. You need to talk to your cluster admin to find out which port range is available to use and confirm that the firewall is opened for the range. You can also set up environment if you want to deploy multiple environments to the same cluster. This is called [environment segregation][]

The above configuration with enableRegistry set to true only works with stand-alone Java instance and Kubernetes cluster with host networking. Currently, docker-compose is not working, and the direct registry is the recommended approach for service discovery. 

* secret.yml

In the secret.yml config file, we need to put the Consul ACL token in so that we can call the Consul API to register and discover the service. 

```
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527

# EmailSender password
emailPassword: change-to-real-password
```

Please note the consulToken defined in the secret.yml example above. 

* client.trustore

Assuming that TLS is used, we need to import the consul certificate to the client.truststore with keytool. You can ask the Consul admin for the certificate. Please refer to [keystore truststore][] for more info. 

* handler.yml

In the handler.yml, we need to define the /health/{serviceId} endpoint to map to a framework implementation of health check handler or your own customized health check. 

```
enabled: true

handlers:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler@exception
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler@metrics
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler@traceability
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler@correlation
  # Swagger Parsing swagger specification based on request uri and method.
  - com.networknt.swagger.SwaggerHandler@specification
  # Security JWT token verification and scope verification (depending on SwaggerHandler)
  - com.networknt.security.JwtVerifyHandler@security
  # Body Parse body based on content type in the header.
  - com.networknt.body.BodyHandler@body
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler@audit
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  # Validator Validate request based on swagger specification (depending on Swagger and Body)
  - com.networknt.validator.ValidatorHandler@validator
  # Health Check Handler
  - com.networknt.health.HealthGetHandler@health
  # Server Info Handler
  - com.networknt.info.ServerInfoGetHandler@info
  # Data handler
  - com.networknt.apid.handler.DataGetHandler@data
chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - specification
    - body
    - audit
    - sanitizer
    - validator

paths:
  - path: '/v1/data'
    method: 'get'
    exec:
      - default
      - data
  - path: '/health/com.networknt.apid-1.0.0'
    method: 'get'
    exec:
      - health
  - path: '/server/info'
    method: 'get'
    exec:
      - info
```

Above /health/{serviceId} endpoint is not protected. If you feel that it needs to be protected and only Consul nodes should access it, you can add IP Whitelist middleware handler in the handler.yml and put it in front of exec: health endpoint. 

When the service register itself to Consul, it will tell Consul to call the /heath/com.networknt.apid-1.0.0 to ensure the service is alive every 10 seconds. The time is configuable for 10 seconds work for most of the scenarios. 

### Deployment

When using Consul registry and discovery in a Kubernetes cluster, all services deployed must use host network so that the service itself can use Kubernetes API to find the IP address and automatically allocate a port number from a range defined in the server.yml config file. 

Here is an example of deployment.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apia-deployment
  labels:
    app: apia
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apia
  template:
    metadata:
      labels:
        app: apia
    spec:
      hostNetwork: true
      containers:
      - name: apia
        image: networknt/com.networknt.apia-1.0.0:1.5.18-ct
        imagePullPolicy: Always
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
```

Please note that hostNetwork is set to true and an env variable "STATUS_HOST_IP" must be passed into the container from a fieldPath called status.hostIP. No static value needs to be put into the deployment config, but it directs the Kubernetes to call its API to get the host ip address and pass it into the container. 


### Downward API

In the Deployment configuration above, the status.hostIP is calling Kubernetes Downward API to get the IP address for the host to allow light-4j application to bind to the host IP address during startup. For Google Cloud Platform, it looks like the internal IP is used when the downward API is called. This is a known issue and we are still working with Google to find a solution. 

### Consul Blocking Queries

For service discovery, we are using Consul Blocking Queries (Long polling). It basically sends a request to the Consul server and tells Consul don't do anything within 10 minutes if the subscribed service instances are not changed. If the subscribed service instances are changed, it will return the result immediately with the changes. After 10 minutes, the request will timeout and a new request will be issued. In this way, we can maintain a list of healthy service instances all the time at the client side. If it is necessary to create a new connection, there is no need to go to Consul for a discovery as the local cache is the latest. This design is the fast way to let the client be notified if any service instance is gracefully shutdown or crashed. 

### Tutorial

There are several tutorials for [service registry and discovery][] and it can help you to understand the details. 


[keystore truststore]: /tutorial/security/keystore-truststore/
[service registry and discovery]: /tutorial/common/discovery/
[Consul Installation Tutorial]: /tool/consul/cluster-install/
[environment segregation]: /design/env-segregation/
