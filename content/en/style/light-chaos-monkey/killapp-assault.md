---
title: "Killapp Assault"
date: 2021-01-28T22:22:46-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When this middleware handler is enabled, it will kill the server during one of the requests. The level in the config file controls the chance of this attack. 

### Config

killapp-assault.yml

```
# Light Chaos Monkey Kill App Assault Handler Configuration.

# Enable the handler if set to true so that it will be wired in the handler chain during the startup
enabled: true
# Bypass the current chaos monkey middleware handler so that attacks won't be triggered.
bypass: true
# How many requests are to be attacked. 1 each request, 5 each 5th request is attacked
level: 5

```

### Expectation

When a particular request triggers the attack, the server instance will shut down. If you are running the server in a docker container, the scheduler should be set up to restart it automatically.
 
For that particular request, no response will be returned. Depending on the client library is used, it might throw an IOException or something else. The client should try to look up another server instance in the cluster and retry the same request. 

When client-side service discovery is used, the client should switch the connection to another instance for all subsequent requests.

In the centralized log, the following info level log statement can be found. 

```
Chaos Monkey - I am killing the Server!
```


### Recovery

If the server runs in a Kubernetes cluster, the scheduler should automatically restart the container.
