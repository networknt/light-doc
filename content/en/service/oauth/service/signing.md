---
title: "Signing"
date: 2019-03-31T13:17:09-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. The payload information can be verified and trusted because it is digitally signed.

As JWTs can be signed you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the token payload, you can also ascertain that the content hasn't been tampered with. The JWT tokens can be shared in the request header, as well as the body of a request/response. It is recommended to share the JWT token in the header.

Most of the times, JWT is used for authorization for service access. However, there is another usage of JWT within the business context for secure information exchange. It is a different use case as it is not related to service security but to address a business-related concern.

In microservices architecture, information might pass through several services before reaching to the target service. Every service between the source service and the target service can modify the information, and there is no way for the target service to ensure that the received info is temper-proofed.

To provide the signing service to users, there are three components. 

* The signing service from the light-oauth2
* An API from the light-4j client module for the signed token issuer
* A validator middleware handler in light-4j to validate the signed token for the receiver

### Signing Server

For signed information exchange, it is simpler than the OAuth 2.0 security. Only three light-oauth2 microservices are needed to fulfill the requirement. 

* Client

For the issuer to access the signing server to get the signed JWT, the issuer must have a valid client_id and client_secret pair to authenticate itself. The client service allows multiple clients to register themselves to get client_id/client_secret. If you only have one or two clients in your organization, you might just prepopulate the client(s) into the database during the deployment and avoid deploying this service. 

* Token

The signing endpoint is part of the token service which is responsible for issuing JWT token in the light-oauth2. Unlike JWT for security, the JWT for secure info exchange has a flexible structure to meet the different requirement from a different organization. It also removed some of the mandatory fields from the security JWT to make it simpler. 

* Key

In a fully distributed microservices architecture, there is no way to copy the public key certificate to each service as there is static IP for the services. The only way is for the service to pull the certificate from the OAuth 2.0 provider the first time the service is started or the first time a new rotated key is received by the service. The key service provides an API for services to get public key certificates when it is necessary. 


### Specification

The following is the signing endpoint specification.

```
  /oauth2/signing:
    post:
      description: Sign a JSON object and return a JWT
      operationId: postSigning
      parameters:
        - name: authorization
          description: encoded client_id and client_secret pair
          in: header
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Successful Operation
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SignRequest'
        description: Signing request object
        required: true

```

And the reference SignRequest is defined in the components/schemas. 

```
    SignRequest:
      type: object
      required:
        - expires
        - payload
      properties:
        expires:
          type: integer
          format: int32
          description: expires in seconds
        payload:
          type: object
          description: payload that needs to be signed
```

As you can see, the caller needs to provide an expiration in seconds, and the payload as a map for the claims. 

### Signing Endpoint

The Oauth2SigningPostHandler.java in the token service handles the signing endpoint. The logic is very similar to Oauth2TokenPostHandler.java which is for JWT access token. 

The following is a list of steps in the handler.


* Verify the client_id and client_secret from the request.
* If client_id/client_secret is authenticated, serialize the body into SignRequest object
* Get the expires from the SignRequest for the JWT expiration
* Add client_id to the claims as well
* Generate a token with claims from SignRequest payload
* Return the token to the caller

### Response

The response is the same as JWT token post handler which is a map with the following keys. 

* access_token - The signed token in JWT
* token_type - "bearer"
* expires_in - The seconds token expires

### Error Messages

The following error messages might be returned if the handler detects an error in the logic. 

* MISSING_AUTHORIZATION_HEADER = "ERR12002";
* INVALID_AUTHORIZATION_HEADER = "ERR12003";
* INVALID_BASIC_CREDENTIALS = "ERR12004";
* CLIENT_NOT_FOUND = "ERR12014";
* UNAUTHORIZED_CLIENT = "ERR12007";
* RUNTIME_EXCEPTION = "ERR10010";
* GENERIC_EXCEPTION = "ERR10014";

### Client Module

In order to access the signing service to get the signed token, the client module has been updated to provide an API to do this automatically. For details, please visit [client module][]. 

### Signed Token Validator

On the receiver service, we provide a middleware handler to verify the token transparently. You need to include the handler into the request/response handler chain and it will verify the signature, expiration and other attributes for the JWT token and then put the payload into an attachment in the exchange to be consumed by the business handler. If the token is not valid in signature or it is already expired, then an error message will be returned instead of calling the business handler. 

For more details, please visit [signature validator][]. 

[client module]: /concern/client/
[signature validator]: /concern/signature-validator/



