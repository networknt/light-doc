---
title: Securing Consul with acl_token
linktitle: Securing Consul with acl_token
date: 2017-10-17T21:03:55-04:00
lastmod: 2018-05-15
description: "Securing Consul by requiring a secret token that will only allow
services who own that token to register thus ensuring discovered instances are valid."
weight: 70
sections_weight: 70
draft: false
toc: true
---

## Introduction

In previous steps, we have set up Consul with acl_default_policy=allow so that
all operations to the Consul server are allowed. This should be only used for internal 
testing. For official environments, we must set acl_default_policy=deny while having
all operations to the Consul server provide an acl_token in the header.

Now let's copy our previous state from `tag` to `token` for each API:
 
```bash
cp -r ~/networknt/discovery/api_a/tag ~/networknt/discovery/api_a/token
cp -r ~/networknt/discovery/api_b/tag ~/networknt/discovery/api_b/token
cp -r ~/networknt/discovery/api_c/tag ~/networknt/discovery/api_c/token
cp -r ~/networknt/discovery/api_d/tag ~/networknt/discovery/api_d/token
```

## Starting Consul

If you consul server is still running, please stop it and restart it with the following:

```bash
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"deny","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0
```

The above command will ensure the default policy is to deny access, and configure the master acl token 
so that services who are privy to the token are able to register.

## Configuring Consul

Create agent token

```bash
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "Name": "Agent Token",
  "Type": "client",
  "Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"read\" }"
}' http://127.0.0.1:8500/v1/acl/create
```

And we get the token like this.

```json
{"ID":"67ccac85-36a3-e912-d0b3-bce1194119a0"}
```

Introduce the token through an API.

```bash
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "Token": "67ccac85-36a3-e912-d0b3-bce1194119a0"
}' http://127.0.0.1:8500/v1/agent/token/acl_agent_token
```


And let's give anonymous users read permissions.

```bash
curl \
    --request PUT \
    --header "X-Consul-Token: the_one_ring" \
    --data \
'{
  "ID": "anonymous",
  "Type": "client",
  "Rules": "node \"\" { policy = \"read\" } service \"\" { policy = \"read\" }"
}' http://127.0.0.1:8500/v1/acl/update
```

It should return something like this:

```json
{"ID":"anonymous"}
```

## Configuring the Servers

The next step we are going to update `secret.yml` to add consul-token in order to access
the Consul for service registry and discovery.

For each server, go ahead and update `secret.yml` to add consulToken: `the_one_ring`

```yaml
consulToken: the_one_ring
```

## Starting the servers

Now let's start four terminals to start servers.  

**API A**

```bash
cd ~/networknt/discovery/api_a/token
mvn clean install -DskipTests exec:exec
```

**API B**

```bash
cd ~/networknt/discovery/api_b/token
mvn clean install -DskipTests exec:exec
```

**API C**

```bash
cd ~/networknt/discovery/api_c/token
mvn clean install -DskipTests exec:exec
```

**API D**

And start the first instance that listen to `7444` as default

```bash
cd ~/networknt/discovery/api_d/token
mvn clean install -DskipTests exec:exec
```

Now you should be able to see the registered service from Consul UI.

```bash
http://localhost:8500/ui
```

In this step, we secured our consul access from services so that we can be sure that
the services registered on the consul are the valid ones. In the next step, we are going
to start services with [Docker][] containers.

[Docker]: /tutorial/common/discovery/docker/
