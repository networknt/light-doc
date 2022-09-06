---
title: "Multiple Specifications"
date: 2022-09-02T11:33:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

For most REST APIs, there is only one specification file openapi.yaml will be configured per instance. With the specification loaded during the startup,  the JWT security can use it to verify the scopes per endpoint, and the schema validator can use it to validate the request and response based on the same specification. 

The above setup covers 99 percent of the use cases in the light-rest-4j applications. However, with light-gateway widely replacing centralized monolithic gateways by our customers, there are two cases where one service instance uses multiple specifications are required.

Case1: The light-gateway is a shared gateway for a group of services, and security verification and request validation are done on the gateway to filter the requests before routing to the downstream APIs. 

Case2: The light-gateway is installed on the same host along with several legacy services to address cross-cutting concerns. In this case, the gateway instance must load all the backend services specifications for security verification and schema validation. 

To enable multiple specs support in the light-rest-4j framework, we have updated the openapi-parse to update that OpenApiHelper to be not a singleton. 

Also, we have updated the OpenApiHandler in the openapi-meta module to load multiple specifications based on the openapi-handler.yml configuration. 

Subsequently, we updated the openapi-security and openapi-validator to leverage the multiple specifications based on the configuration. 


### Configuration

To support multiple specifications, we have added a new config file named openapi-handler.yml

```
# openapi-handler.yml
# This configuration file is used to support multiple OpenAPI specifications in the same light-rest-4j instance.
# An indicator to allow multiple openapi specifications. Default to false which only allow one spec named openapi.yml or openapi.yaml or openapi.json.
multipleSpec: ${openapi-handler.multipleSpec:false}
# Path to spec mapping. One or more base paths can map to the same specifications. The key is the base path and the value is the specification name.
# If users want to use multiple specification files in the same instance, each specification must have a unique base path and it must be set as key.
pathSpecMapping: ${openapi-handler.pathSpecMapping:}
#   v1: openapi-v1
#   v2: openapi-v2

```

By default, the multipleSpec indicator is set to false. So the server is loading only one openapi.yaml by default unless this flag is changed in the values.yml config file to overwrite the default or you can externalize the openapi-handler.yml to the config folder with the property changed to true. 

When the multipleSpec is true, you have to provide a mapping between the basePath to the specification file name without extension. 

Here is an example of openapi-handler.yml that overwrites the default one from the openapi-meta module.

```
# openapi-handler.yml
# This configuration file is used to support multiple OpenAPI specifications in the same light-rest-4j instance.
# An indicator to allow multiple openapi specifications. Default to false which only allow one spec named openapi.yml or openapi.yaml or openapi.json.
multipleSpec: ${openapi-handler.multipleSpec:true}
# Path to spec mapping. One or more base paths can map to the same specifications. The key is the base path and the value is the specification name.
# If users want to use multiple specification files in the same instance, each specification must have a unique base path and it must be set as key.
pathSpecMapping:
  /petstore: openapi-petstore
  /market: openapi-market
```

When using the above config file, you need to make sure that openapi-petstore.yaml and openapi-market.yaml will be in the same folder. In the pathSpecMapping, the key is the basePath which is a common prefix in the requests. This basePath will also be set to the individual helper basePath in case the basePath is not loaded properly from the specification file server section. 

In the single specification case, the basePath will be resolved from the specification server section if possible. Otherwise, it will be overwritten by the basePath configuration property from handler.yml file. Please note that the basePath from handler.yml will only work for single specification. 







