---
title: "Shutdown"
date: 2022-11-08T14:59:29-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

In a production environment, the light-4j server will be started in a Kubernetes cluster or running as a Windows or Linux service. To force the server to restart, you can send a DELETE request to /adm/shutdown endpoint. Once the server is shut down, the K8s will reschedule the pod, and Linux/Windows server will restart the service automatically.

### Specification

The specification is injected to any light-rest-4j application via [openapi-inject.yml](https://github.com/networknt/light-rest-4j/blob/master/openapi-meta/src/main/resources/config/openapi-inject.yml) when the application specific openapi.yaml file is loaded. 


Here is the endpoint definition.

```
  /adm/shutdown:
    delete:
      summary: "Shutdown the service instance to force a restart."
      description: "Returns a JSON body of shutdown time"
      operationId: "serviceShutdown"
      security:
      - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
```

This is a DELETE request, and it requires a token with admin scope if the user belongs to the API platform team or ${server.serviceId}/admin scope if the user owns a specific application. 

### Configuration

If you use light-codegen to generate a light-rest-4j application, the handler.yml will have this endpoint configured. Also, you can find the examples in http-sidecar and light-gateway repositories. 

handler.yml

```
paths:
  .
  .
  .
  - path: '/adm/shutdown'
    method: 'delete'
    exec:
      - admin
      - shutdown
   .
   .
   .

```

If the handler.chains.default is externalized, you need to register the handler class and give an alias to it in values.yml file.

values.yml

```
handler.handlers:
  .
  .
  .
  - com.networknt.server.handler.ServerShutdownHandler@shutdown
```

### Usage

Depending on how the security is configured on the server, you need to get a token and put it into the Authorization header to access this endpoint. If the light portal is used, the request can be sent from the control pane to each individual service instance. When a user logs in to the light portal, a token will be created with the proper scope to access the admin endpoints. 

You can disable the security in the values.yml file for a simple local test. 

values.yml

```
# openapi-security.yml
openapi-security.enableVerifyJwt: false
```
Or skip the security for the /adm/shutdown path in the values.yml

```
# openapi-security.yml
openapi-security.skipPathPrefixes: ["/v1/pets","/adm"]
```
The following is the curl command to send the request directly. 

```
curl -k --location --request DELETE 'https://localhost:8443/adm/shutdown'
```

Once the server receives the request, it will start the shutdown sequence. The response from the server is something like the following. 

```
{"time":1667937042151,"serviceId":"com.networknt.client-gateway-1.0.0"}
```

{{< youtube JG89o0eou2E >}}

