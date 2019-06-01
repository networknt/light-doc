---
title: "Specification"
date: 2017-11-23T14:31:14-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 20	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

First, let's build an OpenAPI 3.0 specification with several endpoints to demo database access. You will need [swagger editor][] to create a specification. Once the specification is created, you can export it into both YAML and JSON format. YAML is easy for designers to work in an editor and is easy to be consumed by the [light-codegen][] and [light-rest-4j][] framework. 

Here is the OpenAPI 3.0 specification created and it can be found in [model-config][] 

```
openapi: 3.0.0
info:
  version: 1.0.0
  title: light-rest-4j Database Tutorial
  description: A demo on how to connect, query and update Oracle/Mysql/Postgres.
  contact:
    email: stevehu@gmail.com
  license:
    name: Apache 2.0
    url: http://www.apache.org/licenses/LICENSE-2.0.html
paths:
  /query:
    get:
      description: Single query to database table
      operationId: getQuery
      responses:
        "200":
          description: successful operation
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/RandomNumber"
              examples:
                response:
                  value:
                    id: 123
                    randomNumber: 456
      security:
        - database_auth:
            - database.r
  /queries:
    get:
      description: Multiple queries to database table
      operationId: getQueries
      parameters:
        - name: queries
          in: query
          description: Number of random numbers
          required: false
          schema:
            type: integer
            format: int32
      responses:
        "200":
          description: successful operation
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/RandomNumber"
              examples:
                response:
                  value:
                    - id: 123
                      randomNumber: 456
                    - id: 567
                      randomNumber: 789
      security:
        - database_auth:
            - database.r
  /updates:
    get:
      description: Multiple updates to database table
      operationId: getUpdates
      parameters:
        - name: queries
          in: query
          description: Number of random numbers
          required: false
          schema:
            type: integer
            format: int32
      responses:
        "200":
          description: successful operation
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/RandomNumber"
              examples:
                response:
                  value:
                    - id: 123
                      randomNumber: 456
                    - id: 567
                      randomNumber: 789
      security:
        - database_auth:
            - database.w
servers:
  - url: http://database.networknt.com/v1
  - url: https://database.networknt.com/v1
components:
  securitySchemes:
    database_auth:
      type: oauth2
      flows:
        implicit:
          authorizationUrl: http://localhost:8888/oauth2/code
          scopes:
            database.w: write database table
            database.r: read database table
  schemas:
    RandomNumber:
      type: object
      required:
        - id
        - randomNumber
      properties:
        id:
          type: integer
          format: int32
          description: a unique id as primary key
        randomNumber:
          type: integer
          format: int32
          description: a random number

```

Now let's clone the model-config repo to your working directory so that we can use the OpenAPI specification as well as the config.json to generate a service project. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```

In the next step, we are going to [generate and build][] a new project. 

[swagger editor]: /tool/swagger-editor/
[light-codegen]: /tool/light-codegen/
[light-rest-4j]: /style/light-rest-4j/
[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi/database
[generate and build]: /tutorial/rest/openapi/database/generation/
