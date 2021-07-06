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
reviewed: true
---

In the previous [Consul TLS][] tutorial, we have set up Consul to ping the service IP address and port number to determine if the service is healthy. It works in certain cases, but not all the cases. It has a flaw as described below.

* A service registers itself to the consul and asks consul to ping the IP and port.
* When the service is crashed without a chance to deregister itself.
* Kubernetes started another service on the same IP and reused the port number as it is dynamically allocated.
* Consul pings the service and it is still alive but it is not the registered service anymore.

We can ask each service to pre-allocate a range of port numbers but it would be very hard to manage or enforce even in the same Kubernetes namespace or Openshift project.

It looks like we cannot use TCP health check in our case although it is the most efficient one.

The TTL based health check is also not ideal as we put a lot of load to the consul servers to update the pass and fail status by calling consul APIs.

The only thing that is left is the HTTP health check option. With the release 1.5.18, we have to update the the /health endpoint to /health/{serviceId} in the new handler.yml so that Consul won't get status code 200 if the target service is not the same service as registered. 

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
# access token to the consul server
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
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
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

As you can see we have changed httpCheck to true and tcpCheck to false. 

In the generated handler.yml config file, double check that the health check path is defined as /health/{serviceId} and skipped all other middleware handlers. 

### Build Docker Image

In order to differentiate the docker image from others for the same API, let's build it as 1.5.18-hh

```
./build.sh 2.0.2-hh
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
        image: networknt/com.networknt.ad-1.0.0:2.0.2-hh
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
https://198.55.49.188:8500/ui/dc1/services/com.networknt.aa-1.0.0
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
