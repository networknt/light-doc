---
title: "Proxy Body"
date: 2021-05-25T13:16:41-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

The light-proxy and light-mesh/http-sidecar use this handler to parse the body into an attachment and keep the stream forwarded to the backend API. It parses the body to a list or map if the content-type is `application/json` in the HTTP header for POST, PUT and PATCH HTTP methods. 

The openapi-validator depends on this handler for parsing the request body streams to JSON and attaching it to exchange to be validated based on the request body schema definition in the specification. 

### body.yml

```
# Enable body parse flag
enabled: ${body.enabled:true}
# cache request body
cacheRequestBody: ${body.cacheRequestBody:false}
# maxBuffers to cache the body for ProxyBodyHandler to read the body. 64K by default.
maxBuffers: ${body.maxBuffers:65536}
```

The ProxyBodyHandler and BodyHandler reside in the same module, and they share the same body.yml configuration file. By default, the body handler is enabled as long as it is added to the handler chain in the handler.yml file. 

A ProxyBodyHandler specification configuration value is called `maxBuffers`, was added, and the default value is 65536 (64K). If your back-end API uses a bigger request, please increase the buffer to make sure it is bigger than the max request size. If you are using the values.yml externalized config file, please add the following entry to customize the buffer size.

```
body.maxBuffers: 1048576
```

### openapi-validator.yml

If you want to use the openapi-validator handler to validate the reqeust body on the proxy, you need to add the ProxyBodyHandler in the handler chain before the openapi-validator, and make sure that the skipBodyValidation is set to false in the openapi-validator.yml file.

```
skipBodyValidation: false
```




