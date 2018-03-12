---
title: "OpenAPI 3.0 Mock"
date: 2018-03-11T19:48:21-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When using light-codegen with OpenAPI 3.0 specification to generate light-rest-4j project,
you can add examples into the specification definition so that the generated project will
have handlers to return corresponding examples.

This is very useful for developers who are working on the service to verify that the service
is working right off the generator. 

Also it is very useful for client side developers as they can get an running service right
off the specification to start their client side application development. Later on they
can switch to the real implementation to do the integration test. 

This kind of support ensures that client side development and server side development can
be running in parallel to speed up the overall delivery process. It is particular useful
with microservivces architecture as each team will only focus on their own service but use
other teams mock services to start the development. 

To leverage this feature in OpenAPI 3.0 specification, you need to define the example in
each endpoint or operation. There are two different format to support single example or
multiple examples.  

Here are serveral example endpoints as part of the [OpenAPI 3.0 petstore specification][]

The following response defines a single example that returns an array of maps. 

```yaml
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
```

The following response defines a single example that returns a map. 

```yaml

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
```

The following response defines multiple examples and the generator will pick up
the first entry to return a map. 

```yaml
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
```


The [light-codegen][] will leverage the defined examples in the generated handler for 
endpoints. If you want to see the example output, please refer to [petstore tutorial][].

Given this feature is built into the light-codegen, it is highly recommended to utilize it
for your API development. It increases productivity dramatically especially in microservices
applications. 

The format of the Swagger 2.0 specification is different than OpenAPI 3.0 specification. If
you are using Swagger 2.0, please refer to [Swagger 2.0 Mock][] tutorial for details. 


[OpenAPI 3.0 petstore specification]: https://github.com/networknt/model-config/blob/master/rest/openapi/petstore/1.0.0/openapi.yaml
[light-codegen]: /tool/light-codegen/
[petstore tutorial]: /tutorial/rest/openapi/petstore/test/
[Swagger 2.0 Mock]: /tutorial/generator/swagger-mock/
