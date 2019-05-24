---
title: "Swagger 2.0 Mock"
date: 2018-03-11T19:48:28-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When using light-codegen with Swagger 2.0 specification to generate light-rest-4j project,
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

To leverage this feature in Swagger 2.0 specification, you need to define the example in
each endpoint or operation. 

Here is an example endpoint as part of the [Swagger 2.0 petstore specification][]

```yaml
      responses:
        '200':
          description: successful operation
          schema:
            $ref: '#/definitions/Pet'
          examples:
            application/json:
              photoUrls:
                - aeiou
              name: doggie
              id: 123456789
              category:
                name: aeiou
                id: 123456789
              tags:
                - name: aeiou
                  id: 123456789
              status: aeiou

```

You can see for this particular endpoint there is a schema defined and also examples defined.

The [light-codegen][] will leverage the defined examples in the generated handler for this
endpoint. If you want to see the example output, please refer to [petstore tutorial][].

Given this feature is built into the light-codegen, it is highly recommended to utilize it
for your API development. It increases productivity dramatically especially in microservices
applications. 

The format of the OpenAPI 3.0 specification is different than Swagger 2.0 specification. If
you are using OpenAPI 3.0, please refer to [OpenAPI 3.0 Mock][] tutorial for details. 


[Swagger 2.0 petstore specification]: https://github.com/networknt/model-config/blob/master/rest/swagger/petstore/2.0.0/swagger.yaml
[light-codegen]: /tool/light-codegen/
[petstore tutorial]: /tutorial/rest/swagger/petstore/test/
[OpenAPI 3.0 Mock]: /tutorial/generator/openapi-mock/
