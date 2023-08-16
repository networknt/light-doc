---
title: "Egress Configuration"
date: 2023-08-15T19:56:09-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

### Caller Backend


When your backend API is calling another API, your backend needs to put the target service_id or service_url as a header in the request. If your network policy supports HTTP, use the HTTP URL; otherwise, use the HTTPS URL. 

For example, you can use the following URL in the request header to access the Petstore Sidecar in the same K8s cluster.

```
service_url: https://petstore.dev-test-ns
 
Or

service_id: com.networknt.petstore-1.0.0
```

Instead of calling the sidecar of the downstream API, call your sidecar with the local host with HTTP protocol in the same pod. 

Assuming that the sidecar is listening to both HTTP and HTTPS on ports 8080 and 8443, we can make the call to the following URL.


```
http://localhost:8080
```


### Caller Sidecar

For most organizations, port 8080 for the sidecar will only be allowed from the Ingress Nginx server. All calls between pods will have to use HTTPS port 8443. Both ports will be mapped to 80 and 443 at the service level in the K8s cluster.


Due to the http-sidecar using a self-signed certificate, we need to turn off the host verification on the caller sidecar in the client.yml section in the values.yml file. 

```
# client.yml
client.verifyHostname: false

```


