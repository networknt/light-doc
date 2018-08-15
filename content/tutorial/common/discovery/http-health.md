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




[Consul TLS]: /tutorial/common/discovery/consul-tls/
