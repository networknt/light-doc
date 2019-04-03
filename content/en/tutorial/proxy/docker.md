---
title: "Docker"
date: 2017-12-16T20:36:56-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In this tutorial, we are going to walk through the dockerized light-proxy in light-docker. There
are two docker-compose files and separate config files for Swagger 2.0 and OpenAPI 3.0 specification.

We are going to use OpenAPI 3.0 compose file and the Swagger 2.0 is very similar with only config
file difference. 

### Start backend services

Before we start the proxy compose, we need to start three instances of OpenAPI proxy-backend services.
The steps are described in [OpenAPI proxy backend][]. Please follow it to start all three instances
and make sure they are working individually. 


[OpenAPI proxy backend]: /tutorial/proxy/openapi-backend/

### Start light-proxy

Now let's clone the light-docker repo from networknt on github.com

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-openapi-proxy.yml up
```

Once the proxy server is up, you can test it with the following curl command.

```
curl -k https://localhost:8080/v1/getData
```

And the result should be something like this.

```json
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8082,"key":"key1"}
```

### proxy.yml

If you want to run this tutorial on your computer, you have to update the proxy.yml in
light-docker/light-proxy/openapi/config folder.

Here is the config file for my computer which is named joy.

```yaml
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: true

# If TLS is enabled when connecting to the target servers
httpsEnabled: true

# Target URIs
hosts: https://joy:8081,https://joy:8082,https://joy:8083

# Connections per thread to the target servers
connectionsPerThread: 20

# Max request time in milliseconds before timeout
maxRequestTime: 10000
```

You need to change the hostname joy to your computer name or your ip address. If
you are using hostname, you need to ensure that you can ping that hostname on your
computer. Otherwise, it is safe to use IP address with 192.xxx.xxx.xxx or 10.xxx.xxx.xxx

### Other config files

There are other config files in the config folder for the docker-compose and you can
modify them to test different behaviour of the light-proxy.

For example, you can enable the security, metrics etc.

