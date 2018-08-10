---
title: Multiple Instances with Environment Segregation
linktitle: Multiple Instances with Environment Segregation
date: 2017-10-17T21:03:06-04:00
lastmod: 2018-05-15
description: "Using tags to denote and differentiate services deployed in the same cluster into logical environments."
weight: 60
sections_weight: 60
draft: false
toc: true
---

## Introduction

In the previous step, we've set up the services to register and discover downstream services through Consul. If there are multiple instances for the same serviceId, then the client will load balance these instances.

In real service delivery, there are a lot of complicated testing scenarios that need to be addressed, and one of the use cases is [environment segregation][]

In this step, we are going to use an environment tag to separate multiple instances during testing so that we can have segregated environments using the same Consul server.


## Configuring the APIs

First, let's copy our current state from the last step into the `tag` directory.

```bash
cp -r ~/networknt/discovery/api_a/consul ~/networknt/discovery/api_a/tag
cp -r ~/networknt/discovery/api_b/consul ~/networknt/discovery/api_b/tag
cp -r ~/networknt/discovery/api_c/consul ~/networknt/discovery/api_c/tag
cp -r ~/networknt/discovery/api_d/consul ~/networknt/discovery/api_d/tag
```

### API A

In order to register the service with a tag to indicate a separate environment. We need to update `server.yml` with an environment tag. Here we are using `test1` as our tag.

```yaml
environment: test1
```

### API B

Let's update `API B` `server.yml` with environment tag.

```yaml
environment: test1
```

### API C

Let's update `API C` `server.yml` with environment tag.

```yaml
environment: test1
```

### API D

Let's update `API D` `server.yml` with environment tag.

```yaml
environment: test1
```

## Starting Consul

Here we are starting the consul server in docker to serve as a centralized registry. To make it simpler, the ACL in consul is disabled by setting acl_default_policy=allow. If you follow the previous step, then your consul server should be running, and you don't need to restart it.

```bash
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

## Starting the Servers

Now let's start four terminals to start servers.  

**API A**

```bash
cd ~/networknt/discovery/api_a/tag
mvn clean install exec:exec
```

**API B**

```bash
cd ~/networknt/discovery/api_b/tag
mvn clean install exec:exec
```

**API C**

```bash
cd ~/networknt/discovery/api_c/tag
mvn clean install exec:exec
```

**API D**

And start the first instance that listen to `7445` as default

```bash
cd ~/networknt/discovery/api_d/tag
mvn clean install exec:exec
```

Now you can see the registered services from Consul UI.

```bash
http://localhost:8500/ui
```

You can find tag `test1` for all services registered on the Consul.


## Testing the Servers

```bash
curl -k https://localhost:7441/v1/data
```

And the result will be

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```
 
## Adding another API
 
Now let's start the second instance of `API D`. Before starting the server, let's update server.yml with port `7445` and with environment tag `test2`.

```yaml
httpsPort:  7445
environment: test2
```

And start the second instance:

```bash
cd ~/networknt/discovery/api_d/tag
mvn clean install exec:exec
```

Take a look at the Consul UI, you can see the tag `test2` for `API D`.

You can use the following command line to lookup API with `test1` or `test2` and you will see one instance returns for each query.

```bash
curl http://localhost:8500/v1/health/service/com.networknt.apid-1.0.0?passing&wait=600s&index=0&tag=test1
curl http://localhost:8500/v1/health/service/com.networknt.apid-1.0.0?passing&wait=600s&index=0&tag=test2
```

Can you guess what will be returned if you don't have tag query parameter? The answer is all available instances will be returned. 


## Testing the added Server

Let's issue the same command again. 

```bash
curl -k https://localhost:7441/v1/data
```

And the result should be the same.

```bash
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Shut down the server with port `7444`. Now when you call the same curl command, you will have an error although instance `7445` server is running.  

This is because that `7445` server has tag `test2` and `API B` is looking up `API D` with tag `test1`.

In summary, this step shows you how to use environment tag to separate testing instances for microservices. 

In the next step, Consul ACL will be enabled so that only services with a [token][] can register and discover from it.

[Consul]: /tutorial/common/discovery/consul/
[environment segregation]: /design/env-segregation/
[token]: /tutorial/common/discovery/token/
