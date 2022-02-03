---
title: "Shared Gateway"
date: 2022-02-03T11:36:20-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

One of the use cases of light-router is to use it as an API gateway for external clients to access internal services. We call it external access points. When light-router is used in this way, there are two deployment patterns. 

* Deploy one instance of light-router [per client][]

* Deploy a share gateway for all exteranl clients

In the light-router repo, we have default config files located in the src/main/resources/config folder; however, they are not suitable for each use case. When using the light-router as a shared gateway, we need to externalize some of the config files to give us the capability to customize according to our customer's requirements. 


In the [config/shared-gateway][] folder, we have added several example configuration files to allow the external client to access the petstore API internally. 

To understand how it works locally, it is recommended to try the pet store API with the light-router in debug mode in IDEs. 

### Petstore

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
git checkout master
cd light-example-4j/rest/petstore-maven-single
mvn clean install -Prelease
java -jar target/server.jar
```

### Gateway

```
cd ~/networknt
git clone https://github.com/networknt/light-router.git
cd light-router
mvn clean install
java -jar -Dlight-4j-config-dir=config/shared-gateway target/light-router.jar
```

### Test

Now, let's send several requests to test the setup. 

Request to the petstore directly. 

```
curl -k https://localhost:9443/v1/pets

[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

Request through the gateway

```
curl -k https://localhost:8443/v1/pets

[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]

```

Health check of the router with no security

```
curl -k https://localhost:8443/health

OK
```

Server info of the router with security

```
curl -k https://localhost:8443/server/info

{"statusCode":401,"code":"ERR10002","message":"MISSING_AUTH_TOKEN","description":"No Authorization header or the token is not bearer type","severity":"ERROR"}
```

[per client](/tutorial/router/client-gateway/)
[config/shared-gateway](https://github.com/networknt/light-router/tree/master/config/shared-gateway)