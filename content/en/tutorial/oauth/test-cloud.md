---
title: "Test Cloud OAuth 2.0 Provider Deployment"
date: 2019-07-26T16:14:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous [form authentication local][] tutorial, we have deployed the light-oauth2 services on the local Ubuntu desktop along with light-router to serve the lightapi.net and signin.lighapi.net single-page applications. 

In this tutorial, we are going to deploy the same configuration to the test cloud so that our developers can utilize the instance for development and testing wherever they can access the Internet.

They allow them to work on both frontend single page applications and the backend services locally without deploy and maintain an instance of light-oauth2 locally. 

### Environment

* Consul Cluster

On our test cloud, we have a [Consul cluster installed][] already, and all light-oauth2 services will be registered there for the light-router instance to look up. 

* Test1 Server

We are going to deploy the seven light-oauth2 services to the test1 VM in the test cloud with docker-compose. To allow other APIs or single-page applications to access light-oauth2 services as SaaS, we need to deploy a light-router instance along with the light-oauth2 services to router the Internet requests. 


* Portal Server

We are going to deploy the light-router instance for the test-portal to the portal server. There are several virtual hosts DNS set up to point to this server from the cloudflare.com.

```
lightapi.net
signin.lightapi.net
```

### Prepare Environment

As we have several servers working together, we need to set up the firewall rules to allow them to communicate with each other. 

The following firewall rules need to be added to the test1 server. 

* test1 needs to allow consul cluster nodes to check health check. 

* test1 needs to allow the portal server to access light-oauth2 services. 

* test1 needs to allow the docker bridge to access the multicast address for the embedded Hazelcast in light-oauth2. 

```
 sudo ufw allow in on br-63a986d0cdce from 172.18.0.1 to 224.2.2.3
```

The following firewall rules need to be added to the three nodes of the consul cluster.

* allow access from test1 server (This is test1 server to test1 server rule to allow multi-cast and it is the only rule that is needed if host network is used for docker-compose)

* allow access from the portal server

The following firewall rules need to be added to the portal server.

* allow 8443, 80, and 443 to access 
* set up routing rule from 443 to 8443 as the portal is listening to 8443, but Internet request is standard HTTPS(443). 

If you have followed the previous tutorial and updated the local desktop /etc/hosts, you need to comment out the following line. Please note that your IP will be different. This will allow you to access the portal server with the Internet DNS. 

```
#192.168.1.144   lightapi.net signin.lightapi.net
```

### Configuration

* light-oauth2

The configuration for the light-oauth2 can be found at https://github.com/networknt/light-config-test/tree/master/light-oauth2/test-cloud, and it is copied from https://github.com/networknt/light-config-test/tree/master/light-oauth2/local-consul in the previous tutorial with some modifications. 

You can compare both folders to find out what has been changed. Here is the summary.

Change the `consulUrl` and `consulToken` in consul.yml to

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# access token to the consul server
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

Change the MySQL host from to the test1 IP address in service.yml file. 

```
jdbcUrl: jdbc:mysql://38.113.162.51:3306/oauth2?useSSL=false&disableMariaDbDriver
```

For the code service only, update the cors.yml to remove the localhost as we will never access from the local. 

```
enabled: false
allowedOrigins:
- https://lightapi.net
- https://signin.lightapi.net
allowedMethods:
- GET
- POST
- PUT
- DELETE
```

Finally, we need to update the docker-compose.yml file to replace the STATUS_HOST_IP for all services. 

```
- STATUS_HOST_IP=38.113.162.51
```

* light-router

This is a light-router instance that exposes all the light-oauth2 services through dev.oauth.lightapi.net as SaaS. Unlike each service listens to different port numbers, the dev.oauth.lightapi.net light-router server will listen to 8443 on HTTPS only. For access from the Internet, a firewall rule will forward all 443 requests to 8443 based on the [iptables configuration](/tutorial/security/port443/).

The configuration for the light-router instance on test1 server along with light-oauth2 services can be found at https://github.com/networknt/light-config-test/tree/master/light-oauth2/test-cloud/light-router

Although the light-oauth2 services are deployed on the same test1 server, we are still using Consul for service lookup to have the flexibility to move the services or deploy more instances on other servers. 

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
# enable HTTP/2 
# must disable when using HTTP with Consul (mostly using local Consul agent), Consul only supports HTTP/1.1 when not using TLS
# optional to enable when using HTTPS with Consul, it will have better performance
enableHttp2: true
```

client.truststore

The client.truststore contains the certificates of the Consul cluster. It is copied from the light-portal configuration. 

client.yml

The light-router instance will need the Http2Client to connect to the Consul for the service lookup and connect to light-oauth2 services as a proxy. As these services are using self-signed certificates with IP addresses only, we need to set the `verifyHostname: false`.

```
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: false
```

cors.yml

The cors will be enabled so that single-page applications can access to the light-oauth2 services from the browser with Authorization code flow. We normally recommend users to develop their react application on https://localhost:3000 and set up a local light-router server to host another login application to interact with the services on test1. The local login server is using a domain name login.lightapi.net which is mapped to the local IP address on the desktop /etc/hosts if Linux desktop is used. 

We also support the open banking demo application and its login app. The final version of the cors.yml looks like the following. 

```
enabled: true
allowedOrigins:
  - https://obsignin.lightapi.net
  - https://login.lightapi.net
  - https://ob.lightapi.net
  - https://localhost:3000
allowedMethods:
  - GET
  - POST
  - PUT
```

handler.yml

The major difference between this instance and light-portal instance is the handler.yml configuration. Here we don't need the StatelessAuthHandler to handle browser interaction like light-portal. The router instance here is just acted as a reverse proxy. 

The handler.yml can be found at https://github.com/networknt/light-config-test/blob/master/light-oauth2/test-cloud/light-router/handler.yml and we have removed some of the services from this router. Only code, token and key services should be exposed here. 

pathPrefixService.yml

Here we map the path to the serviceId for code, token and key services. 

```
# Whether to enable PathPrefixServiceHandler
enabled: true

# mapping from request path prefixes to serviceIds
mapping:
  # Light OAuth 2.0 APIs that is exposed as SaaS
  /oauth2/token: com.networknt.oauth2-token-1.0.0
  /oauth2/code: com.networknt.oauth2-code-1.0.0
  /oauth2/key: com.networknt.oauth2-key-1.0.0

```

The router.yml is the same as the light-portal configuration. 

```
---
# Client Router Configuration
# As this router is built to support discovery and security for light-4j services,
# the outbound connection is always HTTP 2.0 with TLS enabled.

# If HTTP 2.0 protocol will be accepted from incoming request.
http2Enabled: true

# If TLS is enabled when accepting incoming request. Should be true on test and prod.
httpsEnabled: true

# Max request time in milliseconds before timeout to the server.
maxRequestTime: 300000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3
```

In the server.yml, we enable the HTTPS/2 on port 8443 and don't need to register the instance to the Consul. We also need the server.keystore to start the server with HTTPS enabled. 

The following is the service.yml with all the configuration for the service lookup from Consul. 

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
```

* light-portal

The config folder can be found at https://github.com/networknt/light-config-test/tree/master/light-router/test-portal, and it is copied from https://github.com/networknt/light-config-test/tree/master/light-router/local-portal in the previous tutorial. 

After copying, we need to make the following modifications. 

First, we need to update client.truststore to include the certificates to the consul cluster. 

Second, we need to update the client.yml to comment out the server_url for the token service to use the serviceId for the consul lookup. 

Update the consul.yml with the following new URL and token. 

```
# Consul URL for accessing APIs
consulUrl: https://198.55.49.188:8500
# access token to the consul server
consulToken: d08744e7-bb1e-dfbd-7156-07c2d57a0527
```

Update the cors.yml to remove the localhost as there is no need to access the portal from the localhost. 

```
enabled: true
allowedOrigins:
  - https://signin.lightapi.net
  - https://lightapi.net
allowedMethods:
  - GET
  - POST
  - PUT
```

### Start the servers

The sequence to start light-oauth2 and light-router is not important, but we usually start the light-oauth2 first and then light-router. 

* light-oauth2

Log in to test1 server with ssh. 

```
ssh test1
```

Checkout or pull the latest configuration files. 

```
cd ~/networknt/light-config-test
git pull origin master
```

Start the docker-compose

```
cd light-oauth2/test-cloud
docker-compose up -d
```

* light-router

Log in to the portal server with ssh.

```
ssh portal
```

Checkout or pull the latest configuration files.

```
cd ~/networknt/light-config-test
git pull origin master
```

Start the docker-compose

```
cd light-router/test-portal
docker-compose up -d
```

### Test


Now, you can access the portal site lightapi.net from anywhere on the Internet from your browser. 

```
https://lightapi.net
```

Once the lightapi.net is loaded, click the upper-right login button, and you will see that your browser is redirected to the site.

```
https://signin.lightapi.net
```

Fill in the form with 

```
userId: admin
password: 123456
```

And click the SIGN IN button. Your browser will be directed back to the lightapi.net portal site with a JWT token saved into a secure cookie with HTTPS only.   

With the portal UI on lightapi.net and signin.lightapi.net to protect the endpoints of backend services, users can access the backend services through UI securely. 

The instance of light-router on the test1 server along with light-oauth2 services is simple a reverse proxy to ensure that code, token and key services can be accessed from the Internet. These three services all have their own authentication/authorization with client_id and client_secret. 

Other services of light-oauth2 will be exposed by the light-portal to register new client, service, and new users. 

### Summary

The above steps are good for the test environment for development. We will be updating the lightapi.net portal site to add the admin console for the light-oauth2 so that any registered user can create a new client_id/client_secret pair for his/her own application in the development environment. 

[form authentication local]: /tutorial/oauth/form-auth-local/
[Consul cluster installed]: /service/consul/cluster-install/
