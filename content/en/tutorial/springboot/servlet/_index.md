---
title: "Spring Boot Servlet"
date: 2019-03-06T20:57:46-05:00
description: ""
categories: [getting started]
keywords: []
menu:
  docs:
    parent: "getting-started"
    weight: 130
weight: 130
sections_weight: 130
aliases: []
toc: false
draft: false
---

In this tutorial, we are going to generate a typical Spring Boot application from https://start.spring.io/ and modify it to inject light-4j middleware handlers. The final application can be found at https://github.com/networknt/light-example-4j/tree/master/springboot/servlet

Light-4j encourages a design driven approach and it loads the specification during the runtime to validate the request and verify the JWT token scopes based on the specification. For this example application, the specification can be found at https://github.com/networknt/model-config/blob/master/rest/springboot/servlet/openapi.yaml

### Generate Project

Go to https://start.spring.io/ to generate a Spring Boot project. In the form, select the following options. 

* Project: Maven Project
* Language: Java
* Spring Boot: 2.1.3
* Group: com.networknt
* Artifact: demo
* Name: demo
* Description: Demo project for Spring Boot
* Package Name: com.networknt.servlet
* Packaging: Jar
* Java Version: 8
* Dependencies: Web

Once the project is generated, download it to your local and unzip it into your workspace. For me I am using `~/networknt/light-example-4j/springboot/servlet`

### Switch to Undertow

The default embedded server is Tomcat, we need to switch to Undertow so that light-4j handlers can be injected. 

First, we need to exclude tomcat from spring-boot-starter-web. 

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

Next we need to add Undertow starter.

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-undertow</artifactId>
		</dependency>
```

Now, you can start the server and test it with `./mvnw clean install spring-boot:run`. You will find that the web server has switched to the Undertow already from the console. As we don't have any endpoint implemented, there would be an error if you access the root path. 

### Add light-4j spring-servlet

In order to inject the light-4j handlers, we need to add another dependency. 

```
		<dependency>
			<groupId>com.networknt</groupId>
			<artifactId>spring-servlet</artifactId>
			<version>${version.light-4j}</version>
		</dependency>
```

As the dependency refer to the version of the light-4j, we need to define it in the properties section in the pom.xml

```
	<properties>
		<java.version>1.8</java.version>
		<version.light-4j>1.5.32</version.light-4j>
	</properties>
```

The dependency will add typical light-4j middleware handler modules for OpenAPI 3.0 RESTful service. These handlers are defined in the spring-servlet module pom.xml as transitive dependencies. 

### Update DemoApplication

The next step is to update the main application of the Spring Boot to add @ComponentScan("com.networknt"). This allows the application to find the bean defined in the spring-servlet module of light-spring-boot. 

```
package com.networknt.servlet;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@ComponentScan("com.networknt")
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

If your application is not in package `com.networknt`, you need to add another pacakge to the @ComponentScan annotation. 


### Configuration

In this step, we are going to copy the configuration files from the generated OpenAPI petstore application to this project. These config files will be copied to the src/main/resources/config folder for the demo purpose. When deploying the application to the test environment and production, some of the files will externalized. 

The source folder is located at https://github.com/networknt/light-example-4j/tree/master/rest/openapi/petstore/src/main/resources/config

### Update Specification

As the config files are copied from the OpenAPI 3.0 petstore application, it contains all the endpoints to list pets, gets a pet by Id, creates a pet and deletes a pet. We are going to update the specification a little bit to duplicate the endpoints so that we can implement them with both Spring Boot controllers and light-4j handlers.

Here is the updated specification. It is named openapi.json and located in the config folder.

```
openapi: 3.0.0
info:
  version: 1.0.0
  title: Spring Boot and Light-4j Integration
  license:
    name: MIT
servers:
  - url: 'http://springboot.networknt.com'
paths:
  '/light/pets':
    get:
      summary: List all pets
      operationId: lightListPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
      security:
        - petstore_auth:
            - 'read:pets'
      responses:
        '200':
          description: An paged array of pets
          headers:
            x-next:
              description: A link to the next page of responses
              schema:
                type: string
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
              example:
                - id: 1
                  name: catten
                  tag: cat
                - id: 2
                  name: doggy
                  tag: dog
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    post:
      summary: Create a pet
      operationId: lightCreatePets
      requestBody:
        description: Pet to add to the store
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Pet'
      tags:
        - pets
      security:
        - petstore_auth:
            - 'read:pets'
            - 'write:pets'
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  '/light/pets/{petId}':
    get:
      summary: Info for a specific pet
      operationId: lightShowPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      security:
        - petstore_auth:
            - 'read:pets'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              example:
                id: 1
                name: Jessica Right
                tag: pet
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    delete:
      summary: Delete a specific pet
      operationId: lightDeletePetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to delete
          schema:
            type: string
        - name: key
          in: header
          required: true
          description: The key header
          schema:
            type: string
      security:
        - petstore_auth:
            - 'write:pets'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              examples:
                response:
                  value:
                    id: 1
                    name: Jessica Right
                    tag: pet
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  '/spring/pets':
    get:
      summary: List all pets
      operationId: springListPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
          schema:
            type: integer
            format: int32
      security:
        - petstore_auth:
            - 'read:pets'
      responses:
        '200':
          description: An paged array of pets
          headers:
            x-next:
              description: A link to the next page of responses
              schema:
                type: string
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
              example:
                - id: 1
                  name: catten
                  tag: cat
                - id: 2
                  name: doggy
                  tag: dog
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    post:
      summary: Create a pet
      operationId: springCreatePets
      requestBody:
        description: Pet to add to the store
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Pet'
      tags:
        - pets
      security:
        - petstore_auth:
            - 'read:pets'
            - 'write:pets'
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  '/spring/pets/{petId}':
    get:
      summary: Info for a specific pet
      operationId: springShowPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      security:
        - petstore_auth:
            - 'read:pets'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              example:
                id: 1
                name: Jessica Right
                tag: pet
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    delete:
      summary: Delete a specific pet
      operationId: springDeletePetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to delete
          schema:
            type: string
        - name: key
          in: header
          required: true
          description: The key header
          schema:
            type: string
      security:
        - petstore_auth:
            - 'write:pets'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              examples:
                response:
                  value:
                    id: 1
                    name: Jessica Right
                    tag: pet
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  securitySchemes:
    petstore_auth:
      type: oauth2
      description: This API uses OAuth 2 with the client credential grant flow.
      flows:
        clientCredentials:
          tokenUrl: 'https://localhost:6882/token'
          scopes:
            'write:pets': modify pets in your account
            'read:pets': read your pets
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
    Error:
      type: object
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

As you can see, all Spring Boot endpoints have a prefix `/spring` and all Light-4j endpoints have a prefix `light`. 


### Update handler.yml

In light-4j, the handler.yml defines all the handler chains for each endpoint. For each endpoint, you can define a list of middleware handlers in the request/response chain before reaching the final business handler. In the case of Spring Boot, the business handler will be the controller. 

Here is the updated handler.yml

```


# Handler middleware chain configuration
---
enabled: true

#------------------------------------------------------------------------------
# Support individual handler chains for each separate endpoint. It allows framework
# handlers like health check, server info to bypass majority of the middleware handlers
# and allows mixing multiple frameworks like OpenAPI and GraphQL in the same instance.
#
# handlers  --  list of handlers to be used across chains in this microservice
#               including the routing handlers for ALL endpoints
#           --  format: fully qualified handler class name@optional:given name
# chains    --  allows forming of [1..N] chains, which could be wholly or
#               used to form handler chains for each endpoint
#               ex.: default chain below, reused partially across multiple endpoints
# paths     --  list all the paths to be used for routing within the microservice
#           ----  path: the URI for the endpoint (ex.: path: '/v1/pets')
#           ----  method: the operation in use (ex.: 'post')
#           ----  exec: handlers to be executed -- this element forms the list and
#                       the order of execution for the handlers
#
# IMPORTANT NOTES:
# - to avoid executing a handler, it has to be removed/commented out in the chain
#   or change the enabled:boolean to false for a middleware handler configuration.
# - all handlers, routing handler included, are to be listed in the execution chain
# - for consistency, give a name to each handler; it is easier to refer to a name
#   vs a fully qualified class name and is more elegant
# - you can list in chains the fully qualified handler class names, and avoid using the
#   handlers element altogether
#------------------------------------------------------------------------------
handlers:
  # Light-framework cross-cutting concerns implemented in the microservice
  - com.networknt.exception.ExceptionHandler@exception
  - com.networknt.metrics.MetricsHandler@metrics
  - com.networknt.traceability.TraceabilityHandler@traceability
  - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.petstore.handler.PetsGetHandler
  - com.networknt.petstore.handler.PetsPostHandler
  - com.networknt.petstore.handler.PetsPetIdGetHandler
  - com.networknt.petstore.handler.PetsPetIdDeleteHandler

chains:
  default-light:
    - exception
    - metrics
    - traceability
    - correlation
    - specification
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

  default-spring:
    - exception
    - metrics
    - traceability
    - correlation
    - specification
    - security
    # - body
    - audit
    # - dump
    - sanitizer
    - validator

paths:
  - path: '/light/pets'
    method: 'GET'
    exec:
      - default-light
      - com.networknt.petstore.handler.PetsGetHandler

  - path: '/spring/pets'
    method: 'GET'
    exec:
      - default-spring

  - path: '/light/pets'
    method: 'POST'
    exec:
      - default-light
      - com.networknt.petstore.handler.PetsPostHandler
  - path: '/spring/pets'
    method: 'POST'
    exec:
      - default-spring

  - path: '/light/pets/{petId}'
    method: 'GET'
    exec:
      - default-light
      - com.networknt.petstore.handler.PetsPetIdGetHandler
  - path: '/spring/pets/{petId}'
    method: 'GET'
    exec:
      - default-spring

  - path: '/light/pets/{petId}'
    method: 'DELETE'
    exec:
      - default-light
      - com.networknt.petstore.handler.PetsPetIdDeleteHandler
  - path: '/spring/pets/{petId}'
    method: 'DELETE'
    exec:
      - default-spring

  - path: '/health/com.networknt.petstore-3.0.1'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
```

Please note that we have defined two default chains. One is named default-spring for Spring Boot and one is called default-light for Light-4j. The different is that light-4j chain has body parser handler wired in. As Spring Boot controllers need Body stream, we cannot consume the body in the request chain so that body handler is skipped. 

Also, the light-4j endpoint has a final business handler and Spring Boot endpoint only has a middleware handler chain default-spring. The spring-servlet module will automatically call the IntialServletHandler once the handlers in default-spring are all executed. 

### Copy the light-4j handlers

As we have wired in the petstore handlers in the handler.yml file, we need to copy these classes to this project from the petstore project. The source folder is located at https://github.com/networknt/light-example-4j/tree/master/rest/openapi/petstore/src/main/java/com/networknt/petstore/handler

When copying these files, we need to make sure that the package name is the same com.networknt.petstore.handler

These handlers are generated from the light-codegen based on the examples defined in the specification. So they are all mock implementations. 

### Spring Boot Controllers

Now, let’s implement the Spring Boot controllers. We can create a single controller, but for demo purposes, we are going to create three controllers for the four endpoints.

ListPetsController.java

```
package com.networknt.servlet;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ListPetsController {
    @RequestMapping("/spring/pets")
    public String listPets() {
        return "[{\"id\":1,\"name\":\"catten\",\"tag\":\"cat\"},{\"id\":2,\"name\":\"doggy\",\"tag\":\"dog\"}]";
    }
}

```

CreatePetsController.java

```
package com.networknt.servlet;

import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CreatePetsController {
    @RequestMapping(path="/spring/pets", method= RequestMethod.POST)
    public String createPets(@RequestBody String body) {
        return body;
    }
}

```

PetIdController.java

```
package com.networknt.servlet;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PetIdController {
    @RequestMapping("/spring/pets/{petId}")
    public String showPetById(@PathVariable String petId) {
        return "{\"id\":1,\"name\":\"Jessica Right\",\"tag\":\"pet\"}";
    }

    @DeleteMapping("/spring/pets/{petId}")
    public String deletePetById(@PathVariable String petId) {
        return "{\"id\":1,\"name\":\"Jessica Right\",\"tag\":\"pet\"}";
    }
}
```

### Test

Now we have all the handlers and controllers implemented, we can start the server to test the endpoints. We are using curl to test the restful services. 

##### To get a list of pets:

Light-4j endpoint.

```
curl http://localhost:8080/light/pets
```

Spring Boot endpoint.

```
curl http://localhost:8080/spring/pets
```

Both commands will return the same result and the light-4j middleware handlers are all executed. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```


##### To get a list of pets with limit

Light-4j endpoint.

```
curl http://localhost:8080/light/pets?limit=10
```

Spring Boot endpoint.

```
curl http://localhost:8080/spring/pets?limit=10
```

Both commands will return the same result and the light-4j middleware handlers are all executed. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```


##### To get a list of pets with incorrect limit type

Light-4j endpoint.

```
curl http://localhost:8080/light/pets?limit=abc
```

Spring Boot endpoint.

```
curl http://localhost:8080/spring/pets?limit=abc
```

In the specification, we have defined the limit as integer. When `abc` is used, both commands will return the same error as light-4j [openapi-validator](https://doc.networknt.com/style/light-rest-4j/openapi-validator/) is injected at low level http core. It invokes [json-schema-validator](https://github.com/networknt/json-schema-validator) for the query parameter limit.

```
{"statusCode":400,"code":"ERR11004","message":"VALIDATOR_SCHEMA","description":"Schema Validation Error - $: string found, integer expected","severity":"ERROR"}
```

##### To get a single pet by Id

Light-4j endpoint.

```
curl http://localhost:8080/light/pets/111
```

Spring Boot endpoint.

```
curl http://localhost:8080/spring/pets/111
```

Both commands will return the same result and the light-4j middleware handlers are all executed. 

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

##### To create a new pet with Post request

Light-4j endpoint.

```
curl -X POST http://localhost:8080/light/pets -H 'Content-Type: application/json' -d '{"id":1,"name":"Jessica Right","tag":"pet"}'
```

Spring Boot endpoint.

```
curl -X POST http://localhost:8080/spring/pets -H 'Content-Type: application/json' -d '{"id":1,"name":"Jessica Right","tag":"pet"}'
```

Both commands will return the same result and the light-4j middleware handlers are all executed. 

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

Note that the body validation is disabled in the openapi-validator.yml as we have to pass the body stream to the Spring Boot. 


##### To delete a pet


Light-4j endpoint.

```
curl -X DELETE http://localhost:8080/light/pets/1 -H 'key: 1'
```

Spring Boot endpoint.

```
curl -X DELETE http://localhost:8080/spring/pets/1 -H 'key: 1'
```

Both commands will return the same result and the light-4j middleware handlers are all executed. 

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

Note that there is an extra header `key: 1` in the request as the specification for this endpoint require a key header. You can try to remove the key header and see the validation error as below. 

```
{"statusCode":400,"code":"ERR11017","message":"VALIDATOR_REQUEST_PARAMETER_HEADER_MISSING","description":"Header parameter key is required on path /light/pets/{petId} but not found in request.","severity":"ERROR"}
```

### Security

For the above test, you can see that request validation handler based on the specification is working. In fact, there are other handlers like exception, metrics, tracebility, correction, sanitizer and security wired in and working behind the scene. In the configuration, the JWT token verfication is disable, let's enable it and to check if the security will work for both Spring Boot and Light-4j endpoints. 

Let's open openapi-security.yml and change the enableVerifyJwt to true and rebuild and restart the server.

```
./mvnw clean install spring-boot:run
```

Now, if we issue the following commands which worked before. 

```
curl http://localhost:8080/light/pets
curl http://localhost:8080/spring/pets
```

You will receive the same error as below. 

```
{"statusCode":401,"code":"ERR10002","message":"MISSING_AUTH_TOKEN","description":"No Authorization header or the token is not bearer type","severity":"ERROR"}
```

In order to access the service, you need to include a bear JWT token in the request header. 

Here is the updated commands with a long lived token in the header with the scopes match the openapi.yaml specification. The token can be found in the [petstore tutotial][].

Spring Boot

```
curl -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" http://localhost:8080/spring/pets

```

And you will get the normal result. 

```
[{"id":1,"name":"catten","tag":"cat"},{"id":2,"name":"doggy","tag":"dog"}]
```

[petstore tutotial]: /tutorial/rest/openapi/petstore/security/
