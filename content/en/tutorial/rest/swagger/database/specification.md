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

First, let's build a Swagger 2.0 specification with several endpoints to demo database access. You will need [swagger editor][] to create a specification. Once the specification is created, you can export it into both YAML and JSON format. YAML is easy for designers to work in an editor and JSON is easy to be consumed by the [light-codegen][] and [light-rest-4j][] framework. 

Here is the Swagger 2.0 specification created and it can be found in [model-config][] 

```
swagger: '2.0'

info:
  version: "1.0.0"
  title: light-rest-4j Database Tutorial
  description: A demo on how to connect, query and update Oracle/Mysql/Postgres. 
  contact:
    email: stevehu@gmail.com
  license:
    name: "Apache 2.0"
    url: "http://www.apache.org/licenses/LICENSE-2.0.html"
host: database.networknt.com
schemes:
  - http
  - https
basePath: /v1

consumes:
  - application/json
produces:
  - application/json

paths:
  /query:
    get:
      description: Single query to database table
      operationId: getQuery
      responses:
        200:
          description: "successful operation"
          schema:
            $ref: "#/definitions/RandomNumber"          
      security:
      - database_auth:
        - "database.r"
  /queries:
    get:
      description: Multiple queries to database table
      operationId: getQueries
      parameters:
      - name: "queries"
        in: "query"
        description: "Number of random numbers"
        required: false
        type: "integer"
        format: "int32"
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/RandomNumber"
      security:
      - database_auth:
        - "database.r"
  /updates:
    get:
      description: Multiple updates to database table
      operationId: getUpdates
      parameters:
      - name: "queries"
        in: "query"
        description: "Number of random numbers"
        required: false
        type: "integer"
        format: "int32"
      responses:
        200:
          description: "successful operation"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/RandomNumber"
      security:
      - database_auth:
        - "database.w"
securityDefinitions:
  database_auth:
    type: "oauth2"
    authorizationUrl: "http://localhost:8888/oauth2/code"
    flow: "implicit"
    scopes:
      database.w: "write database table"
      database.r: "read database table"
definitions:
  RandomNumber:
    type: "object"
    required:
    - "id"
    - "randomNumber"
    properties:
      id:
        type: "integer"
        format: "int32"
        description: "a unique id as primary key"
      randomNumber:
        type: "integer"
        format: "int32"
        description: "a random number"
```

Now let's clone the model-config repo to your working directory so that we can use the Swagger specification as well as the config.json to generate a service project. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
```

In the next step, we are going to [generate and build][] a new project. 

[swagger editor]: /tool/swagger-editor/
[light-codegen]: /tool/light-codegen/
[light-rest-4j]: /style/light-rest-4j/
[model-config]: https://github.com/networknt/model-config/tree/master/rest/swagger/database
[generate and build]: /tutorial/rest/swagger/database/generation/
