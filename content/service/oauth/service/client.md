---
title: "Client Registration"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 40
aliases: []
toc: false
draft: false
---

Before initiating the protocol, the client registers with the authorization server. The 
means through which the client registers with the authorization server are not defined
in OAuth 2.0 specification. 

As an extension, we have implemented client registration/on-boarding as a micro service
that exposes several endpoints. 

Unlike other OAuth 2.0 providers, we have extend the client registration to define the
relationship between client and service (service registers itself) and service's endpoints.

Before digging into the details of implementation, let's clarify some concepts about
client. 

### Client Type

OAuth defines two client types, based on their ability to authenticate securely with 
the authorization server (i.e., ability to maintain the confidentiality of their 
client credentials):

* confidential

Clients capable of maintaining the confidentiality of their credentials (e.g., client 
implemented on a secure server with restricted access to the client credentials), or 
capable of secure client authentication using other means.

* public

Clients incapable of maintaining the confidentiality of their credentials (e.g., 
clients executing on the device used by the resource owner, such as an installed 
native application or a web browser-based application), and incapable of secure 
client authentication via any other means.

Above are standard client types defined in the specification and we have added
another one to control which client can issue resource owner password credentials
grant request. 

* trusted

These clients are marked as trusted and they are the only clients that can issue
resource owner password credentials grant type. For API management team, please
make sure that trusted client is also confidential and the client and resource
must be deployed and managed by the same organization as this flow is not as
secure as authorization code and client credentials flows.


The client type designation is based on the authorization server's definition of 
secure authentication and its acceptable exposure levels of client credentials.  
The authorization server does not make assumptions about the client type.

A client may be implemented as a distributed set of components or services, each
with a different client type and security context (e.g., a distributed client with 
both a confidential server-based component and a public browser-based component).  
In this case, the client should register each component or service as a separate 
client.

In a microservices architecture, a service might call other services to fulfill
its request, in this case, it should register itself as a service and a client.
That means the owner needs to follow both service on-boarding and client on-boarding
processes.

### Client Profile

This specification has been designed around the following client profiles:

* web application (web server)

A web application is a confidential client running on a web server. Resource owners 
access the client via an HTML user interface rendered in a user-agent on the device 
used by the resource owner. The client credentials as well as any access token issued 
to the client are stored on the web server and are not exposed to or accessible by 
the resource owner.

* user-agent-based application (browser)
      
A user-agent-based application is a public client in which the client code is 
downloaded from a web server and executes within a user-agent (e.g., web browser) on 
the device used by the resource owner. Protocol data and credentials are easily 
accessible (and often visible) to the resource owner. Since such applications
reside within the user-agent, they can make seamless use of the user-agent 
capabilities when requesting authorization.

* native application (mobile)

A native application is a public client installed and executed on the device used by 
the resource owner. Protocol data and credentials are accessible to the resource owner.  
It is assumed that any client authentication credentials included in the application can 
be extracted. On the other hand, dynamically issued credentials such as access tokens or 
refresh tokens can receive an acceptable level of protection. At a minimum, these 
credentials are protected from hostile servers with which the application may interact.  
On some platforms, these credentials might be protected from other applications residing 
on the same device.

The specification only mentioned above client profiles and the following two profiles
are added in our OAuth 2.0 implementation.

* batch application (batch)

Batch jobs are very similar with web application but they are managed by enterprise
scheduler and executed in a projected environment. It is considered as confidential
client.

* service (service)

Services are usually protected as resources but in a microservices architecture, a
service can also be a client to call other services or resources. These services
normally running within light-weight containers in a secured environment. And they
are considered as confidential clients.

### Client Identifier

The authorization server issues the registered client a client identifier - a unique 
string representing the registration information provided by the client. The client 
identifier is not a secret; it is exposed to the resource owner and MUST NOT be used
alone for client authentication. The client identifier is unique to the authorization 
server. In our implementation, it is a UUID generated on the server. Here is an example:

```
f7d42348-c647-4efb-a52d-4c5787421e72
```
   
### Client Secret

Clients in possession of a client secret MAY use the HTTP Basic authentication scheme 
as defined in [RFC2617] to authenticate with the authorization server. The client 
identifier is encoded using the "application/x-www-form-urlencoded" encoding algorithm 
, and the encoded value is used as the username; the client secret is encoded using 
the same algorithm and used as the password. The authorization server supports the HTTP 
Basic authentication scheme for authenticating clients that were issued a client secret.

For example (with extra line breaks for display purposes only):

```
     Authorization: Basic czZCaGRSa3F0Mzo3RmpmcDBaQnIxS3REUmJuZlZkbUl3
```

   
### Other Authentication Methods
   
Currently we don't support other authentication method but we are open to support others
if there are request from our users.

### Unregistered Clients

Due to security reasons, all client must be registered before authenticated on the server.
Unregistered clients are not supported on this implementation.

### Client Micro Service

This service has several endpoints and listening to port 6884.

Here is the specification.

```yaml
swagger: '2.0'
info:
  version: 1.0.0
  title: OAuth2 Client Registration
  description: OAuth2 Client Registration microservices endpoints.
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
  /oauth2/client:
    post:
      description: Return a client object
      operationId: createClient
      parameters:
        - in: body
          name: body
          description: Client object that needs to be added
          required: true
          schema:
            $ref: '#/definitions/Client'
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Client'
      security:
        - client_auth:
            - oauth.client.w
    put:
      description: Return the updated client
      operationId: updateClient
      parameters:
        - in: body
          name: body
          description: Client object that needs to be added
          required: true
          schema:
            $ref: '#/definitions/Client'
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Client'
      security:
        - client_auth:
            - oauth.client.w
    get:
      description: Return all clients
      operationId: getAllClient
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
        - name: clientName
          in: query
          description: Partial clientName for filter
          required: false
          type: string
      responses:
        '200':
          description: successful operation
          schema:
            type: array
            items:
              $ref: '#/definitions/Client'
      security:
        - client_auth:
            - oauth.client.r
  '/oauth2/client/{clientId}':
    delete:
      description: Delete a client by Id
      operationId: deleteClient
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
      responses:
        '400':
          description: Invalid clientId supplied
        '404':
          description: Client not found
      security:
        - client_auth:
            - oauth.client.w
    get:
      description: Get a client by Id
      operationId: getClient
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
          schema:
            $ref: '#/definitions/Client'
        '400':
          description: Invalid clientId supplied
        '404':
          description: Client not found
      security:
        - client_auth:
            - oauth.client.r
            - oauth.client.w
  '/oauth2/client/{clientId}/service':
    delete:
      description: Delete all associated services for a client by clientId
      operationId: deleteAllClientService
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: Client not found
      security:
        - client_auth:
            - oauth.client.w
    get:
      description: Get all associated services and endpoints by clientId
      operationId: getAllClientService
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: Client not found
      security:
        - client_auth:
            - oauth.client.r
            - oauth.client.w
  '/oauth2/client/{clientId}/service/{serviceId}':
    post:
      description: Link a service and its endpoints to a client
      operationId: linkClientService
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
        - name: body
          in: body
          description: A list of endpoints that needs to be linked to the client
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: Client or service not found
      security:
        - client_auth:
            - oauth.client.w
    delete:
      description: Delete all endpoints of a service for a client
      operationId: deleteClientService
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: Client or service not found
      security:
        - client_auth:
            - oauth.client.w
    get:
      description: Get linked endpoints of a service from a client
      operationId: getClientService
      parameters:
        - name: clientId
          in: path
          description: Client Id
          required: true
          type: string
        - name: serviceId
          in: path
          description: Service Id
          required: true
          type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: Client or service not found
      security:
        - client_auth:
            - oauth.client.r
            - oauth.client.w
securityDefinitions:
  client_auth:
    type: oauth2
    authorizationUrl: 'http://localhost:8888/oauth2/code'
    flow: implicit
    scopes:
      oauth.client.w: write oauth client
      oauth.client.r: read oauth client
definitions:
  Client:
    type: object
    required:
      - clientType
      - clientProfile
      - clientName
      - clientDesc
      - ownerId
      - scope
    properties:
      clientId:
        type: string
        description: a unique client id
      clientSecret:
        type: string
        description: client secret
      clientType:
        type: string
        description: client type
        enum:
          - confidential
          - public
          - trusted
      clientProfile:
        type: string
        description: client profile
        enum:
          - webserver
          - browser
          - mobile
          - service
          - batch
      clientName:
        type: string
        description: client name
      clientDesc:
        type: string
        description: client description
      ownerId:
        type: string
        description: client owner id
      scope:
        type: string
        description: client scope separated by space
      redirectUri:
        type: string
        description: redirect uri

```

#### /oauth2/client/{clientId}@delete

This endpoint is used to delete existing client. The following validation will be
performed in the service.

* If clientId cannot be found in the in-memory grid, then the following error will
be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
```

#### /oauth2/client/{clientId}@get

This endpoint is used to get a particular client with clientId. The following
validation will be performed in the service.

* If clientId cannot be found in the in-memory grid, then the following error will
be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
```


#### /oauth2/client@get

This endpoint gets all the clients from client service with filter and sorted on 
clientName. A page query parameter is mandatory and pageSize and clientName filter
are optional.

* page 

Page number which must be specified. It starts with 1 and an empty list will
be returned if the page is greater than the last page.

* pageSize

Default pageSize is 10 and you can overwrite it with another number. Please don't
use a big number due to performance reason. 

* clientName

This is the only filter available and it supports filter by start with a few characters.
For example, "clientName=abc" means any clientName starts with "abc". The result is also
sorted by clientName in the pagination. 


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


#### /oauth2/client@post

This endpoint is used to create a new client. This usually will be called from light-portal
and the following validations will be performed before a new client is added.

* Verify that clientId exist in the cache. The clientId is generated as a UUID so this
cannot be triggered. It is implemented this way just want to be in sync with other
services.

```
  "ERR12019": {
    "statusCode": 400,
    "code": "ERR12019",
    "message": "CLIENT_ID_EXISTS",
    "description": "Client id %s exists."
  }
```

* Verify that ownerId is in user cache in memory. If it doesn't exist, the following
error will be returned.

```
  "ERR12013": {
    "statusCode": 404,
    "code": "ERR12013",
    "message": "USER_NOT_FOUND",
    "description": "User %s is not found."
  }
```

* Make sure the clientType is from a list of valid values. If not, an error message
will be returned.

```
  "ERR11004": {
    "statusCode": 400,
    "code": "ERR11004",
    "message": "VALIDATOR_SCHEMA",
    "description": "Schema Validation Error - %s"
  }
```

* Make sure that clientProfile is from a list of valid values. If not, an error
message will be returned.

```
  "ERR11004": {
    "statusCode": 400,
    "code": "ERR11004",
    "message": "VALIDATOR_SCHEMA",
    "description": "Schema Validation Error - %s"
  }
```


#### /oauth2/client@put

This endpoint is used to update an existing client. This usually will be called from 
light-portal and the following validations will be performed before a client is updated.

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
```

* Verify that ownerId is in user cache in memory. If it doesn't exist, the following
error will be returned.

```
  "ERR12013": {
    "statusCode": 404,
    "code": "ERR12013",
    "message": "USER_NOT_FOUND",
    "description": "User %s is not found."
  }
```

* Make sure the clientType is from a list of valid values. If not, an error message
will be returned.

```
  "ERR11004": {
    "statusCode": 400,
    "code": "ERR11004",
    "message": "VALIDATOR_SCHEMA",
    "description": "Schema Validation Error - %s"
  }
```

* Make sure that clientProfile is from a list of valid values. If not, an error
message will be returned.

```
  "ERR11004": {
    "statusCode": 400,
    "code": "ERR11004",
    "message": "VALIDATOR_SCHEMA",
    "description": "Schema Validation Error - %s"
  }
```

#### /oauth2/client/{clientId}/service@get

This endpoint get all the linked services and endpoints for a particular client. 

Here is an example of response body. 

```json
{"AACT0001":["/v1/data@get"]}
```

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
```

#### /oauth2/client/{clientId}/service@delete

This endpoint deletes all services and endpoints for a particular client. It is very dangerous
API and must be careful in using it. It is supposed to be used only when a client is retired. 

After the links are removed, the client's scope will be recalculated and normally means there is
no scope for this client anymore. 

The response body will contains the old scope and new scope for the client. Here is an example.

```json
{"old_scope":"petstore.r petstore.w","new_scope":""}
```

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
```


#### /oauth2/client/{clientId}/service/{serviceId}@post

This endpoint adds one or more endpoints of a service to a client. It removes all existing links
and insert new ones. Once the insert is done, the client's scope is recalculated based on the
updated info. 

The response body will contains the old scope and new scope for the client. Here is an example.

```json
{"old_scope":"petstore.r petstore.w","new_scope":"data.w data.r"}
```

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
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

#### /oauth2/client/{clientId}/service/{serviceId}@delete

This endpoint deletes endpoints for a service that is linked to a particular client. Once the
action is done, it will recalculate scope for the client to reflect the changes. 

The response body will contains the old scope and new scope for the client. Here is an example.

```json
{"old_scope":"petstore.r petstore.w","new_scope":""}
```

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
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


#### /oauth2/client/{clientId}/service/{serviceId}@get

This endpoint gets a list of endpoints of a service linked to a particular client. 

The response body will contains a list of linked endpoints. Here is an example.

```json
["/v1/data@get"]
```

* Verify that clientId exist in the cache. If clientId doesn't existing the cache, an
error message will be returned.

```
  "ERR12014": {
    "statusCode": 404,
    "code": "ERR12014",
    "message": "CLIENT_NOT_FOUND",
    "description": "Client %s is not found."
  }
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


