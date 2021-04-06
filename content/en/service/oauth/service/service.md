---
title: "Service Registration"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 30
aliases: []
toc: false
draft: false
reviewed: true
---

Every microservice or API needs to register itself to OAuth2 server in order to control who can access it. During the registration/on-boarding, a list of scopes defined in the OpenAPI specification or other schema contracts for GraphQL and Hybrid RPC should be populated as well. This list of scopes will be used for clients to register scopes in order to access this particular service or API during client registration.

This service has several endpoints and listens to port 6883.

### specification.

```yaml
swagger: '2.0'
info:
  version: 1.0.0
  title: OAuth2 Service Registration
  description: OAuth2 Service Registration microservices endpoints.
  contact:
    email: stevehu@gmail.com
  license:
    name: Apache 2.0
    url: 'http://www.apache.org/licenses/LICENSE-2.0.html'
host: oauth2.networknt.com
schemes:
  - http
  - https
consumes:
  - application/json
produces:
  - application/json
paths:
  /oauth2/service:
    post:
      description: Return a service object
      operationId: createService
      parameters:
        - in: body
          name: body
          description: Service object that needs to be added
          required: true
          schema:
            $ref: '#/definitions/Service'
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Service'
      security:
        - service_auth:
            - oauth.service.w
    put:
      description: Return the updated service
      operationId: updateService
      parameters:
        - in: body
          name: body
          description: Service object that needs to be added
          required: true
          schema:
            $ref: '#/definitions/Service'
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Service'
      security:
        - service_auth:
            - oauth.service.w
    get:
      description: Return all services
      operationId: getAllService
      parameters:
        - name: page
          in: query
          description: Page number
          required: true
          type: integer
          format: int32
        - name: pageSize
          in: query
          description: Pag size
          required: false
          type: integer
          format: int32
        - name: serviceId
          in: query
          description: Partial serviceId for filter
          required: false
          type: string
      responses:
        '200':
          description: successful operation
          schema:
            type: array
            items:
              $ref: '#/definitions/Service'
      security:
        - service_auth:
            - oauth.service.r
  '/oauth2/service/{serviceId}':
    delete:
      description: Delete a service by Id
      operationId: deleteService
      parameters:
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '400':
          description: Invalid serviceId supplied
        '404':
          description: Service not found
      security:
        - service_auth:
            - oauth.service.w
    get:
      description: Get a service by Id
      operationId: getService
      parameters:
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Service'
        '400':
          description: Invalid serviceId supplied
        '404':
          description: Service not found
      security:
        - service_auth:
            - oauth.service.r
            - oauth.service.w
  '/oauth2/service/{serviceId}/endpoint':
    post:
      description: create endpoints for service
      operationId: createServiceEndpoint
      parameters:
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
        - in: body
          name: body
          description: A list of endpoint object
          required: true
          schema:
            type: array
            items:
              $ref: '#/definitions/ServiceEndpoint'
      responses:
        '201':
          description: Successful response
      security:
        - service_auth:
            - oauth.service.w
    delete:
      description: Delete all endpoints for a service
      operationId: deleteServiceEndpoint
      parameters:
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '400':
          description: Invalid serviceId supplied
        '404':
          description: Service not found
      security:
        - service_auth:
            - oauth.service.w
    get:
      description: Get all endpoints for a service
      operationId: getServiceEndpoint
      parameters:
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
          schema:
            type: array
            items:
              $ref: '#/definitions/ServiceEndpoint'
        '400':
          description: Invalid serviceId supplied
        '404':
          description: ServiceEndpoint not found
      security:
        - service_auth:
            - oauth.service.r
            - oauth.service.w
securityDefinitions:
  service_auth:
    type: oauth2
    authorizationUrl: 'http://localhost:8888/oauth2/code'
    flow: implicit
    scopes:
      oauth.service.w: write oauth service
      oauth.service.r: read oauth service
definitions:
  Service:
    type: object
    required:
      - serviceId
      - serviceName
      - serviceType
      - scope
    properties:
      serviceId:
        type: string
        description: a unique service id
      serviceType:
        type: string
        description: service type
        enum:
          - swagger
          - openapi
          - graphql
          - hybrid
      serviceName:
        type: string
        description: service name
      serviceDesc:
        type: string
        description: service description
      ownerId:
        type: string
        description: service owner userId
      scope:
        type: string
        description: service scopes separated by space
      createDt:
        type: string
        format: date-time
        description: create date time
      updateDt:
        type: string
        format: date-time
        description: update date time
  ServiceEndpoint:
    type: object
    required:
      - endpoint
      - operation
      - scope
    properties:
      endpoint:
        type: string
        description: a combination of path and method to uniquely identify an operation
      operation:
        type: string
        description: operationId of the endpoint
      scope:
        type: string
        description: scope associated with the endpoint

```

### /oauth2/service@get

This endpoint gets all the services from service with filter and sorted on 
serviceId. A page query parameter is mandatory and pageSize and serviceId filter
are optional.

* page 

Page number which must be specified. It starts with 1 and an empty list will
be returned if the page is greater than the last page.

* pageSize

Default pageSize is 10, and you can overwrite it with another number. Please don’t use a big number due to performance reasons.

* serviceId

This is the only filter available and it supports filters by starting with a few characters. For example, “serviceId=abc” means any serviceId starts with “abc”. The result is also sorted by serviceId in the pagination.

The following validation will be performed in the service.

* If page is missing from the query parameter, an error will be returned.

```
  "ERR11000": {
    "statusCode": 400,
    "code": "ERR11000",
    "message": "VALIDATOR_REQUEST_PARAMETER_QUERY_MISSING",
    "description": "Query parameter '%s' is required on path '%s' but not found in request."
  }
```

### /oauth2/service@post

This endpoint is used to create a new service. The following validation will be performed
before a new service is created.

* If serviceId exists in the cache, then the following error will be returned.

```
  "ERR12018": {
    "statusCode": 400,
    "code": "ERR12018",
    "message": "SERVICE_ID_EXISTS",
    "description": "Service id %s exists."
  }
```

* If ownerId doesn't exist in user cache, then the following error will be returned.

```
  "ERR12013": {
    "statusCode": 404,
    "code": "ERR12013",
    "message": "USER_NOT_FOUND",
    "description": "User %s is not found."
  }
```


### /oauth2/service@put

This is the endpoint to update existing services. Before service is updated, the following validation will be performed.

* If serviceId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12015",
    "message": "SERVICE_NOT_FOUND",
    "description": "Service %s is not found."
  }
```

* If ownerId doesn't exist in user cache, then the following error will be returned.

```
  "ERR12013": {
    "statusCode": 404,
    "code": "ERR12013",
    "message": "USER_NOT_FOUND",
    "description": "User %s is not found."
  }
```

### /oauth2/service/{serviceId}@delete

This endpoint is used to delete existing services. Before the service is deleted, the following validations will be performed.

* If serviceId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12015",
    "message": "SERVICE_NOT_FOUND",
    "description": "Service %s is not found."
  }
```

### /oauth2/service/{serviceId}@get

This endpoint is used to get a particular service by serviceId. Before the service is
returned, the following validation will be performed.

* If serviceId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12015",
    "message": "SERVICE_NOT_FOUND",
    "description": "Service %s is not found."
  }
```

### /oauth2/service/{serviceId}/endpoint@post

This is an endpoint that is used to register endpoints for a particular service. As you can
see there is no put method for this resource so all endpoints will be deleted and then insert
if this endpoint is called. 

Here is an example of request body.

```json
[{"endpoint":"/v1/data@post","operation":"createData","scope":"data.w"},{"endpoint":"/v1/data@put","operation":"updateData","scope":"data.w"},{"endpoint":"/v1/data@get","operation":"retrieveData","scope":"data.r"},{"endpoint":"/v1/data@delete","operation":"deleteData","scope":"data.w"}]
```

* If serviceId cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12015",
    "message": "SERVICE_NOT_FOUND",
    "description": "Service %s is not found."
  }
```

### /oauth2/service/{serviceId}/endpoint@delete

This is an endpoint that is used to delete all the endpoints for a particular service. You have to make sure that there are no client links to the service endpoints before issuing a delete request. Otherwise, you will have database constraint errors.

* If service endpoints cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12042",
    "message": "SERVICE_ENDPOINT_NOT_FOUND",
    "description": "Service %s is not found."
  }
```


### /oauth2/service/{serviceId}/endpoint@get

This is an endpoint that is used to get all the endpoints for a particular service. 

Here is an example of response body

```json
[{"factoryId":5,"id":5,"operation":"retrieve data","endpoint":"/v1/getData@get","scope":"data.r"},{"factoryId":5,"id":5,"operation":"create data","endpoint":"/v1/postData@post","scope":"data.w"}]
```

* If service endpoints cannot be found in the service cache, then the following error will be
returned.

```
  "ERR12015": {
    "statusCode": 404,
    "code": "ERR12042",
    "message": "SERVICE_ENDPOINT_NOT_FOUND",
    "description": "Service %s is not found."
  }
```
