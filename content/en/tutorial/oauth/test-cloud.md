---
title: "Test Cloud Deployment"
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

### Environment

* Consul Cluster

On our test cloud, we have a [Consul cluster installed][] already, and all light-oauth2 services will be registered there for the light-router instance to look up. 

* Test1 Server

We are going to deploy the seven light-oauth2 services to the test1 VM in the test cloud with docker-compose. 

* Portal Server

We are going to deploy the light-router instance for the test-portal to the portal server. There are several virtual hosts DNS set up to point to this server from the cloudflare.com

```
lightapi.net
signin.lightapi.net
```

### Prepare Environment

As we have several servers working together, we need to set up the firewall rules to allow them to communicate with each other. 

The following firewall rules need to be added on the test1 server. 

* test1 needs to allow consul cluster nodes to check health check. 

* test1 needs to allow porta server to access light-oauth2 services. 

* test1 needs to allow the docker bridge to access the multicast address for the embedded Hazelcast in light-oauth2. 


```
 sudo ufw allow in on br-63a986d0cdce from 172.18.0.1 to 224.2.2.3
```

The following firewall rules need to be added to the three nodes of the consul cluster.

* allow access from test1 server
* allow access from the portal server

The following firewall rules need to be added to the portal server

* allow 8443, 80 and 443 to access 
* set up routing rule from 443 to 8443 as the portal is listening to 8443, but Internet request is standard HTTPS(443). 

If you have followed the previous tutorial and updated the /etc/hosts, you need to comment out the following line. Please note that your IP will be different. 

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

Change the mysql host from to the test1 IP address in service.yml file. 

```
jdbcUrl: jdbc:mysql://38.113.162.51:3306/oauth2?useSSL=false&disableMariaDbDriver
```

For code service only, update the cors.yml to remove the localhost as we will never access from the local. 

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

Once the lightapi.net is loaded, click the upper-right login button and you will see that your browser is redirected to the site.

```
https://signin.lightapi.net
```

Fill in the form with 

```
userId: admin
password: 123456
```

And click the SIGN IN button. Your browser will be directed back to the lightapi.net portal site with a JWT token saved into a secure cookie with HTTPS only.   

### Summary

The above steps are good for the test environment for development. We will be updating the lightapi.net portal site to add admin console for the light-oauth2 so that any registered user can create a new client_id/client_secret pair for his/her own application in the development environment. 


[form authentication local]: /tutorial/oauth/form-auth-local/
[Consul cluster installed]: /service/consul/cluster-install/
