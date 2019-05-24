---
title: "Start Local Light Router"
date: 2019-05-01T15:38:25-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [started the light-oauth2][] services locally. Now let's start the light-router instance to be responsible for service discovery and serving the resources later. 

For development, we are going to use the proxy from the create-react-app to connect to the light-router for all API accesses. Once the development is done, the SPA application will be built and copied to a folder named lightapi which match the domain lightapi.net

To start the local light-router instance, we are going to use a docker-compose.

```
cd ~/networknt/light-config-test/light-router/local-portal
docker-compose up -d
```

The above commands will start a light-router container, and it will listen to port 8443. The local-portal folder above is almost identical as light-portal in the same parent folder with the following modifications. 

### client.yml

As we are starting the light-oauth2 services with Docker containers without Let's Encrypt certificates that match the domain names. In the light-router configuration folder, we need to update the client.yml to disable the `verfyHostname`. 

```
verifyHostname: false
```

### consul.yml

The `consulUrl` is changed from the static IP to the lightapi.net host and switched from `https` to `http`.  

```
consulUrl: http://lightapi.net:8500
```

### docker-compose.yml

In the docker-compose.yml, we are going to map port 443 to container port 8443. In a production deployment, we usually use `iptables` to map the ports instead of exposing 443 from the Docker. 

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    networks:
    - localnet
    ports:
    - 443:8443
    volumes:
    - ./config:/config
    - ./faucet/build:/faucet/build
    - ./webclient/build:/webclient/build
    - ./lightapi/build:/lightapi/build
    - ./taiji/build:/taiji/build

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true

```

After the router instance is started, you should have access to the virtual host site https://lightapi.net

### Test

To ensure that the light-oauth2 APIs can be accessed from the router, let's test with light-oauth2 client service.

Calling the client service directly. 

```
curl -k https://lightapi.net:6884/oauth2/client?page=1
```

Calling the client service through light-router.

```
curl -k https://lightapi.net/oauth2/client?page=1
```

Both above commands should have the same response. 

```
[{"clientId":"f7d42348-c647-4efb-a52d-4c5787421e72","clientSecret":null,"clientType":"public","clientProfile":"mobile","clientName":"PetStore Web Server","clientDesc":"PetStore Web Server that calls PetStore API","ownerId":"admin","scope":"petstore.r petstore.w","customClaim":"{\"c1\": \"361\", \"c2\": \"67\"}","redirectUri":"http://localhost:8080/authorization","authenticateClass":null,"derefClientId":null}]
```

At this moment, all backend services are up and running. Let's [start portal view][] SPA from the light-portal repository in the next step. 

[started the light-oauth2]: /tutorial/portal/local-router/light-oauth2/
[start portal view]: /tutorial/portal/local-router/portal-view/
