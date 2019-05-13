---
title: "Specification"
date: 2018-03-11T19:00:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Light-rest-4j microservices framework encourages Design Driven API building and 
[OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification) is the central
piece to drive the runtime for security and validation. Also, the specification 
can be used to scaffold a running server project the first time so that developers 
can focus their efforts on the domain business logic implementation without 
worrying about how each component are wired together.

During the service implementation phase, specification might be changed and you can
regenerate the service codebase again without overwriting your handlers and test
cases for handlers. The regeneration is also useful if you want to upgrade to the
latest version of the frameworks for your project. 

To create OpenAPI specification, the best tool is [swagger-editor][] and I have an 
[article][] in tool section to describe how to use it.

By following the instructions on how to use the editor, let's create four API specifications 
in model-config repository using OpenAPI 3.0 specification standard. 

API AA will call API AB, API AC and API AD

```
API AA -> API AB 
      -> API AC 
      -> API AD
```

Here is the API AA openapi.yaml and others can be found at
https://github.com/networknt/model-config/tree/master/rest/openapi/aa/1.0.0 
or model-config/rest/openapi folder in your workspace as we've checked it out in
[preparation][] step.  

```yaml
openapi: 3.0.0
servers:
  - url: 'http://aa.networknt.com/v1'
info:
  version: 1.0.0
  title: API AA for aggregate microservices pattern
  description: >-
    API AA is called by consumer directly and it will call API AB, API AC and API AD to get data
  contact:
    email: stevehu@gmail.com
  license:
    name: Apache 2.0
    url: 'http://www.apache.org/licenses/LICENSE-2.0.html'
paths:
  /data:
    get:
      description: Return an array of strings collected from down stream APIs
      operationId: listData
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                title: ArrayOfStrings
                type: array
                items:
                  type: string
              examples:
                response:
                  value:
                    - Message 1
                    - Message 2
      security:
        - aa_auth:
            - aa.w
            - aa.r
components:
  securitySchemes:
    aa_auth:
      type: oauth2
      flows:
        implicit:
          authorizationUrl: 'http://localhost:8080/oauth2/code'
          scopes:
            aa.w: write access
            aa.r: read access
```

As defined in the specification, API AA will return a list of stings and it requires
scope aa.r or scope aa.w to access the endpoint /data.

In the next step, we are going to use light-codegen to [scaffold][] four projects. 

[swagger-editor]: https://swagger.io/swagger-editor/
[article]: /tool/swagger-editor/
[preparation]: /tutorial/rest/openapi/ms-aggregate/preparation/
[scaffold]: /tutorial/rest/openapi/ms-aggregate/generation/

