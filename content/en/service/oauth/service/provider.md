---
title: "Provider Registration"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 30
aliases: []
toc: false
draft: false
---

The service to support federated OAuth 2.0 providers.

For example, external OAuth 2.0 provider for external clients and internal OAuth 2.0 provider for internal clients.


 It is a federation protocol aimed at simplifying authorization and access to protected data by giving access to data while protecting the owner’s account credentials.
 It allows a user with an account on one website (the service provider) to allow another website (the consumer) to access his or her data from the first website.



This service has several endpoints and listens to port 6889.

### specification.

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: OAuth2 provider Registration
  license:
    name: MIT
servers:
  - url: 'http://light-oauth2.swagger.io'
paths:
  /oauth2/provider:
    post:
      summary: 'Registe a new oauth provider '
      operationId: createProvider
      requestBody:
        description: provider object
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Provider'
      tags:
        - providers
      security:
        - provider_auth:
            - 'read:provider'
            - write.provider
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Provider'
        '201':
          description: Null response
    put:
      summary: 'Update oauth provider '
      operationId: updateProvider
      requestBody:
        description: provider object
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Provider'
      tags:
        - providers
      security:
        - provider_auth:
            - 'read:provider'
            - write.provider
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Provider'
        '201':
          description: Null response
    get:
      summary: 'Return list of registed oauth provider '
      operationId: getProviders
      tags:
        - providers
      security:
        - provider_auth:
            - 'read:provider'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Provider'
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  '/oauth2/provider/{providerId}':
    delete:
      summary: delete a provider by providerId
      operationId: deleteProviderById
      tags:
        - providers
      parameters:
        - name: providerId
          in: path
          required: true
          description: The id of the provider
          schema:
            type: string
      security:
        - provider_auth:
            - 'read:provider'
            - 'write:provider'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Provider'
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
components:
  securitySchemes:
    provider_auth:
      type: oauth2
      description: This API uses OAuth 2 with the client credential grant flow.
      flows:
        clientCredentials:
          tokenUrl: 'https://localhost:6882/token'
          scopes:
            'write:provider': modify provider
            'read:provider': read provider
  schemas:
    Provider:
      required:
        - serverUrl
      properties:
        providerId:
          type: string
        serverUrl:
          type: string
        uri:
          type: string
        providerName:
          type: string
    Error:
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string

```

### /oauth2/provider@get

This endpoint gets all the providers from service




### /oauth2/provider@post

This endpoint is used to create a new provider. The following validation will be performed
before a new provider is created.

* If providerId exists in the cache, it means the providerId has been registered, then the following error will be returned.

```
  "ERR12048": {
    "statusCode": 400,
    "code": "ERR12048",
    "message": "PROVIDER_ID_EXISTS",
    "description": "Provider id %s exists; It has been regristed already."
  }
```




### /oauth2/provider@put

This is the endpoint to update existing provider. Before provider is updated, the
following validation will be performed.

* If providerId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12049": {
    "statusCode": 404,
    "code": "ERR12049",
    "message": "PROVIDER_ID_INVALID",
    "description": "Provider id invalid."
  }
```



### /oauth2/provider/{providerId}@delete

This endpoint is used to delete existing provider. Before the provider is deleted,
the following validations will be performed.

* If serviceId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12047": {
    "statusCode": 404,
    "code": "ERR12047",
    "message": "PROVIDER_ID_NOT_EXISTING",
    "description": "The provider id is not existing"
  }
```
