---
title: "Specification"
date: 2017-11-29T10:32:34-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


Light*4j microservices frameworks encourage Design Driven API building and for REST
style of APIs we support Swagger 2.0 specification and OpenAPI 3.0 specification. In
this tutorial, we are going to use Swagger 2.0 specification as the central piece to 
drive the runtime for security and validation. Also, the specification can be used to 
scaffold a running server project the first time so that developers can focus their 
efforts on the domain business logic implementation without worrying about how each 
component are wired together.

During the service implementation phase, specification might be changed and you can
regenerate the service codebase again without overwriting your handlers and test
cases for handlers. The regeneration is also useful if you want to upgrade to the
latest version of the frameworks for your project. 

To create Swagger 2.0 specification, the best tool is [swagger-editor][] and I have an
[article][] in tool section to describe how to use it. 

By following the instructions on how to use the editor, let's create four API specifications 
in model-config repository.

API A will call API B, API B will call API C, API C will call API D

```
API A -> API B -> API C -> API D
```

Here is the API A swagger.yaml and others can be found at [model-config][] or 
model-config/rest/swagger folder in your workspace. 

```
swagger: '2.0'

info:
  version: "1.0.0"
  title: API A for microservices demo
  description: API A is called by consumer directly and it will call API B and API C to get data
  contact:
    email: stevehu@gmail.com
  license:
    name: "Apache 2.0"
    url: "http://www.apache.org/licenses/LICENSE-2.0.html"
host: a.networknt.com
schemes:
  - http
basePath: /v1

consumes:
  - application/json
produces:
  - application/json

paths:
  /data:
    get:
      description: Return an array of strings collected from down stream APIs
      operationId: listData
      responses:
        200:
          description: Successful response
          schema:
            title: ArrayOfStrings
            type: array
            items:
              type: string
          examples: {
            "application/json": ["Message 1","Message 2"]
          }
      security:
        - a_auth:
          - api_a.w
          - api_a.r

securityDefinitions:
  a_auth:
    type: oauth2
    authorizationUrl: http://localhost:8080/oauth2/code
    flow: implicit
    scopes:
      api_a.w: write access
      api_a.r: read access
```

As defined in the specification, API A will return a list of stings and it requires
scope api_a.r or scope api_a.w to access the endpoint /data.

The next step, we will build the generator and generate the projects based on the specs.

[Swagger 2.0 specification]: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md
[OpenAPI 3.0 specification]: https://swagger.io/specification/
[swagger-editor]: http://swagger.io/swagger-editor/
[article]: /tool/swagger-editor/
[model-config]: https://github.com/networknt/model-config/tree/master/rest/swagger