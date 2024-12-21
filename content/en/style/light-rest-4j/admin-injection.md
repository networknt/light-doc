---
title: "Admin Injection"
date: 2022-10-07T17:27:11-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

All light-4j applications will have the following admin endpoints injected during the startup. 

```
openapi: 3.0.0
paths:
  /adm/health/${server.serviceId:com.networknt.placeholder-1.0.0}:
    get:
      description: pass through to the liveness endpoint that belongs to backend api
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin
  /adm/server/info:
    get:
      description: get the proxy server info and corresponding backend service info
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin

  /adm/chaosmonkey:
    get:
      description: to get the current chaosmonkey settings
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin
  /adm/chaosmonkey/{assault}:
    post:
      description: to update chaosmonkey settings
      requestBody:
        description: to update chaosmonkey settings
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: true
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin

  /adm/logger:
    get:
      description: to get the current logging settings
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin
    post:
      description: to modify the logging settings
      requestBody:
        description: to update logging settings
        content:
          application/json:
            schema:
              type: "array"
              items:
                type: "object"
                properties:
                  name:
                    type: "string"
                  level:
                    type: "string"
        required: true
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin

  /adm/logger/content:
    get:
      description: to get the content of the logs from this service.
      security:
        - admin-scope:
            - admin
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin

  /adm/modules:
    get:
      description: to get the all registered modules and handlers to form a dropdown for module config reload
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin
    post:
      description: to trigger the config reload for all modules or specified modules selected on control pane
      requestBody:
        description: to trigger the reload from the config server from the control pane.
        required: true
        content:
          application/json:
            schema:
              type: array
              items:
                type: string
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin
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
            - ${server.serviceId:com.networknt.placeholder-1.0.0}/admin

components:
  securitySchemes:
    admin-scope:
      type: oauth2
      description: This API uses OAuth 2 with the client credential grant flow.
      flows:
        clientCredentials:
          scopes:
            admin: admin scope to access admin endpoints
```

This config file is located in the light-rest-4j/openapi-meta/src/main/resources/config folder.

### Health Check

The endpoint is a get method at path /adm/health/${server.serviceId} to return the status of the health for the application. Normal light-4j applications use the HealthGetHandler in the [health][] module to return the health status. For http-sidecar and light-gateway, the [proxy health][] implementation is ProxyHealthGetHandler in the ingress-proxy module, and it can check the backend's health status by calling the backend health check endpoint. Both implementations share the same health.yml configuration. 

### Server Info

The endpoint is a get method at path /adm/server/info to return all the server runtime information. It is the best place to check your final configurations for the server.

When you change your configuration for a module and reload it through the [config-reload][] API, you can check the server info before and after reloading to ensure the configuration for the module is changed. 



[health]: /concern/health/
[proxy health]: /concern/proxy-health/
[config-reload]: /concern/config-reload/
