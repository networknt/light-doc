---
title: "Skip Scope Verification"
date: 2022-12-11T16:39:28-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This is a newly added flag to the openapi-security.yml for the light-gateway to skip the JWT scope verification for the backend APIs that don't have specifications or specifications that are not deployed to the light-gateway instance. 

On a centralized gateway, users usually will enable security to ensure all the backend APIs requests are valid. Also, the security handler will protect the admin endpoints injected from the openapi-inject.yml file. Those admin endpoints have admin or {serviceId}/admin scope and need scope verification to be enabled in the openapi-security.yml configuration file. 

As we know, the OpenAPI specification must be present in the config folder to support the scope verification, as the scopes are defined in the spec, and the JWT scope needs to be compared with the scope in the endpoint specification. 

For some gateway deployments, not all OpenAPI specifications are deployed to the gateway. With scope verification enabled, all access to the backend API without specification will receive an error message to indicate the request path is invalid because the specification for the endpoint cannot be found.

When the security handler tries to verify the scope, it gets the operation from the specification and fails. That is why an error message is in the response. To skip the JWT scope verification when the spec doesn't exist, you can set skipVerifyScopeWithoutSpec to true. 

This allows the security handler to validate the signature and expiration of the JWT token only for the backend API endpoints. Of course, if one backend API has the specification deployed to the gateway, then the handler will verify the scope. 


### Configuration

For the security handler to leverage the specification for the endpoint, we need to parse the specification before the security handler in the request/response chain. Here is the default chain definition for the handler.yml in values.yml file.


```
handler.chains.default:
  - exception
  # - metrics
  # - limit
  - traceability
  - correlation
  # - cors
  # - stateless
  - limit
  - specification
  - security
  .
  .
  .

```

When we enable the specification handler in the default chain as above, we need to update the openapi-handler.yml to ignore the invalid path error when the endpoint cannot be found in the specification or the specification doesn't exist. Otherwise, you will receive an error from the specification handler, and the security handler cannot be reached. 


```
# openapi-handler.yml
openapi-handler.ignoreInvalidPath: true
```

Finally, we want to skip the scope verification in the security handler if the specification cannot be found. Here is the property from the openapi-security.yml that we can overwrite in the values.yml file.


```
# openapi-security.yml
openapi-security.skipVerifyScopeWithoutSpec: true
```

With the above configuration, the security handler will verify the scope for all the admin endpoints and endpoints you have specification files deployed to the gateway. For other backend API endpoints, only the JWT  signature and expiration will be verified. The scope verification will be done at the backend API with the http-sidecar or light-proxy-server(LPS), depending on the deployment architecture. 


