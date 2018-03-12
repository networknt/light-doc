---
title: "Router Assisted Service Discovery"
date: 2018-03-05T10:47:40-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

[Light-router][] is a service that provides consumers another option to do service discovery
if they cannot leverage [client][] module provided by light-4j.

There are two ways to [deploy light-router][] and it is highly recommended for client to own
the instance of light-router.  

For demo purpose, the light-router will be deployed in Kubernetes cluster master node as it
has a static IP address. By following the steps below, you should have a router instance up
and running and connect to multiple instances of API A in the Kubernetes cluster. Communication
between API A, B, C and D is handled with client module inside each API. 

### Environment

The light-router will be deployed to sandbox which is our development Kubernetes cluster master
node. 


### Config

All configuration can be found at https://github.com/networknt/light-config-test/tree/master/light-router/cloud/config

### Start 

To start the server there is a docker-compose file in https://github.com/networknt/light-config-test/tree/master/light-router/cloud

The content of the docker-compose file:

```yaml
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    networks:
    - localnet
    ports:
    - 8080:8080
    volumes:
    - ./config:/config

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true

```

You must start the docker-compose at light-config-test/tree/master/light-router/cloud folder.

```
docker-compose up -d
``` 


### Test

Before we start using the light-router, let's make sure that we can access the reference apis
with API A. 

Go to the consul server http://38.113.162.50:8500/ to find an instance of API A. Here is an
example.  

```
curl -k https://38.113.162.51:2406/v1/data
```

And the following result should be expected. 

```json
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```

Now let's access the router instance with the following command line. 

```
curl -k -H "service_id: com.networknt.apia-1.0.0" https://38.113.162.50:8080/v1/data
```

And the result is:

```json
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```


[Light-router]: /service/router/
[client]: /concern/client/
[deploy light-router]: /service/router/location-ownership/
