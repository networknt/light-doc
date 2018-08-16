---
title: "Consul HTTP Health Check"
date: 2018-08-14T22:47:35-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [Consul TLS][] tutorial, we have set up Consul to ping the service IP address and port number to determine if the service is healthy. It works in certain cases, but not all the cases. It has a flaw as described below.

* A service registers itself to the consul and asks consul to ping the IP and port.
* When the service is crashed without a chance to deregister itself.
* Kubernetes started another service on the same IP and reused the port number as it is dynamically allocated.
* Consul pings the service and it is still alive but it is not the registered service anymore.

We can ask each service to pre-allocate a range of port numbers but it would be very hard to manage or enforce even in the same Kubernetes namespace or Openshift project.

It looks like we cannot use TCP health check in our case although it is the most efficient one.

The TTL based health check is also not ideal as we put a lot of load to the consul servers to update the pass and fail status by calling consul APIs.

The only thing that is left is the HTTP health check option. With the release 1.5.18, we have update the the /health endpoint to /health/{serviceId} in the new handler.yml so that Consul won't get status code 200 if the target service is not the same service as registered. 

### Create HTTP health folder

Now let's copy from consul-tls to http-health for each API.
 
```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r consul-tls http-health
cd ~/networknt/light-example-4j/discovery/api_b
cp -r consul-tls http-health
cd ~/networknt/light-example-4j/discovery/api_c
cp -r consul-tls http-health
cd ~/networknt/light-example-4j/discovery/api_d
cp -r consul-tls http-health
```

### API D

Let's update API D configuration files to switch to multiple chains and HTTP health check. 

consul.yml

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

As you can see we have changed httpCheck to true and tcpCheck to false. 

service.yml

```

# Singleton service factory configuration/IoC injection
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
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.apid.PathHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook

```

As you can see that we have remove the entire middleware handler section. 

handler.yml

```
---
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

This is a brand new config file that is used to support multiple chains. As you can see that the health check path is defined as /health/{serviceId} and skipped all other middleware handlers. 

### Build Docker Image

In order to differentiate the docker image from others for the same API, let's build it as 1.5.18-hh

```
./build.sh 1.5.18-hh
```

### Update Deployment

Once the docker image is built, let's update the apid-deployment.yml to with the new image name. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apid-deployment
  labels:
    app: apid
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apid
  template:
    metadata:
      labels:
        app: apid
    spec:
      hostNetwork: true
      containers:
      - name: apid
        image: networknt/com.networknt.apid-1.0.0:1.5.18-hh
        imagePullPolicy: Always
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP

```

### Deployment

Now let's check in all the updates to the API D and login to the Kubernetes master node. 

On the k8s master ndoe. 

```
cd ~/networknt/light-example-4j
git pull origin develop
cd discovery/api_d/http-health/
kubectl create -f apid-deployment.yaml
```

### Consul UI

You can check the Consul UI from the following url. Please note that you need the ACL token to access the Consul UI. 

```
https://198.55.49.188:8500/ui/dc1/services
```

### API C, B, A

Make the similar updates for other APIs and deploy them. 

### Test

Now let's go to the consul UI to find an instance of api_a. 

```
https://198.55.49.188:8500/ui/dc1/services/com.networknt.apia-1.0.0
```

Click the instance and find out the IP and port number. Now, let's construct a curl command to test the service chain. 

```
curl -k https://38.113.162.53:2406/v1/data
```

The result should be something like the following. 

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

### Summary

In this tutorial, we have walked through the configurations of the four services to work with a Consul cluster that supports TLS and ACL. Also, we have updated the consul.yml to switch from tcpCheck to httpCheck. All config files are updated internally and packaged into the Docker container. In the next step, we are going to use the externalized config files in [External Config][] tutorial.


[Consul TLS]: /tutorial/common/discovery/consul-tls/
[External Config]: /tutorial/common/discovery/external-config/
