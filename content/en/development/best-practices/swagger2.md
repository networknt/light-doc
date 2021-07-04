---
title: "Swagger 2.0 Specification Best Practices"
date: 2018-03-17T05:36:31-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Swagger 2.0 specification is a very loose specification that gives designer many options to write the spec. In most cases, developers write the code with annotations and generate the specification afterward. With an enterprise
scale in mind, we encourage a [design first][] approach. The outcome is not just a document but a specification that
can be used to scaffold a new project and loaded during runtime to verify JWT scopes for security and validate requests
to protect the business layer against attacks.

To enable [light-codegen][] to generate meaningful code and utilize the full potential of the [light-rest-4j][] framework,
the author of the OpenAPI 3.0 specification should follow the best practices below.

### Guidelines

- clear and easily readable by architects, analysts, developers
- well documented, with explanations provided in description tags
- adheres to OpenAPI specifications - v2.0 at the time of this writing
- uses a design which lends itself to a clean and easily consumable object model

### Security First

Often API designers focus on functionalities and add security later on. We would encourage to follow security first
approach so that security is considered for every endpoint during the design.

Before working on a new specification, you can copy from an existing one and [petstore][] is a good starting point.
There are also other Swagger specifications in the [model-config][] repository to help you in learning the design style.   

Within petstore specification you can find the following block that defines the security under the component. This section
is optional in the specification, but here it is mandatory. [light-codegen][] throws an error if securityDefinitions are not
found.  

```yaml
securityDefinitions:
  petstore_auth:
    type: oauth2
    authorizationUrl: 'http://petstore.swagger.io/oauth/dialog'
    flow: implicit
    scopes:
      'write:pets': modify pets in your account
      'read:pets': read your pets
```

Once the securityDefinitions is defined, you can specify security scopes for each endpoint. When you add a new endpoint you
might ask yourselves a question. Do I need to create a brand new scope or pick one or more existing scopes from the scopes
defined in securityDefinitions?

The endpoint/operation definition with security definition looks like this.

```yaml
    delete:
      tags:
        - pet
      summary: Deletes a pet
      description: ''
      operationId: deletePet
      produces:
        - application/xml
        - application/json
      parameters:
        - name: api_key
          in: header
          required: false
          type: string
        - name: petId
          in: path
          description: Pet id to delete
          required: true
          type: integer
          format: int64
      responses:
        '400':
          description: Invalid ID supplied
        '404':
          description: Pet not found
      security:
        - petstore_auth:
            - 'write:pets'
            - 'read:pets'

```

Above design ensures that the [light-rest-4j][] framework compares the scopes from JWT token against the scopes defined
for each endpoint at runtime. It gives you the flexibility to grant permissions based on endpoints to consumers.

### Definitions

In the specification, models define what would be the request/response body for each endpoint. There are two different
places to define models. Flattened/scattered in each endpoint or extracted into the definitions.

When the model definition scattered in each endpoint, there is no name and the same model might be duplicated in several
endpoints. The generator does not generate POJO classes as it does not make sense to generate some classes named body0,
body1, body2, etc. Chances are body0 is for get request, and body2 is for put request. As both of them are dealing a
same set of attributes, they are the same. How would developers remember that in your get handler, you should use body0
but in your put handler, you should use body2? They would be shocked when they found that these two classes have the
same fields and methods except the class names are different.

To generate POJO classes with proper names and to avoid duplications, it is best to extract common data objects into the
definitions section.

Here is an example in the petstore specification.

```yaml
definitions:
  Order:
    type: object
    properties:
      id:
        type: integer
        format: int64
      petId:
        type: integer
        format: int64
      quantity:
        type: integer
        format: int32
      shipDate:
        type: string
        format: date-time
      status:
        type: string
        description: Order Status
        enum:
          - placed
          - approved
          - delivered
      complete:
        type: boolean
        default: false
    xml:
      name: Order
```

The above Order definition is used to generate a Java class called Order in [light-codegen][]. With Order is defined, you
can put $ref in each point for object definition. Here is an example response that utilizes the Order definition.

```yaml
  /store/order:
    post:
      tags:
        - store
      summary: Place an order for a pet
      description: ''
      operationId: placeOrder
      produces:
        - application/xml
        - application/json
      parameters:
        - in: body
          name: body
          description: order placed for purchasing the pet
          required: true
          schema:
            $ref: '#/definitions/Order'
      responses:
        '200':
          description: successful operation
          schema:
            $ref: '#/definitions/Order'
        '400':
          description: Invalid Order

```

Here is another response example with an array.   

```yaml
  /user/createWithArray:
    post:
      tags:
        - user
      summary: Creates list of users with given input array
      description: ''
      operationId: createUsersWithArrayInput
      produces:
        - application/xml
        - application/json
      parameters:
        - in: body
          name: body
          description: List of user object
          required: true
          schema:
            type: array
            items:
              $ref: '#/definitions/User'
      responses:
        default:
          description: successful operation

```

### Examples

Swagger specification gives you an opportunity to define an example response for each endpoint so that your API consumer
can easily understand what would be expected when the endpoint is accessed. Also, [light-codegen][] has a feature to
generate mock responses based on the examples defined in the specification. Once you have examples defined, the generated
project can be built and started with mock responses for consumers to start their work immediately without waiting for
the provider to complete the API implementation.

For more details on this topic, please refer to [Swagger 2.0 Mock][].


### Naming Convention

As definition name is translated into Java class name, it is better to follow the naming convention of Java.

- Elements should always start with an upper-case letter, to respect class definitions in generated code, which always start with an upper-case letter

```
  properties:
     error:
       $ref: 'org/APIERR0001/1.0.1/errors.yaml#/error'
  ...
  vs
  ...
  properties:
    error:
      $ref: 'org/APIERR0001/1.0.1/errors.yaml#/Error'
```

- Elements should use only alpha-numeric characters and avoid underscores, @ signs or others. OpenAPI general guidelines recommend alpha-numeric only, and, while these would generate correct programming language code, it would break accepted programming guidelines.

```
  "@nameID":
    $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/NameID"
  "@accessCode":
    $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/Access_Code"
...
  filterList:
     type: array
     items:
       $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/APIFilter_Type"
...
vs
...
  nameID:
    $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/NameID"
  accessCode:
    $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/AccessCode"
...
  filterList:
    type: array
    items:
      $ref: "org/APIDOMAIN0001/0.0.5/elementDefs.yaml#/APIFilterType"    
```

### Object definitions are to be avoided from the declaration of Body elements

The objects should be moved to the __*definitions*__ section of the specification or in an external document, for __*shared definitions*__.

The Body should not contain any declarations as in "type: object".

__Ex.: *selection element*__:

__Original version:__

```
  ...
  selection:
    description:  Identifies the selection to retrieve information for. Only one of the child elements of this structure are to be provided.
    type: object
    properties:
      id:
        $ref: "org/APIDOMAIN0001/0.0.1/elementDefs.yaml#/ID"
      card:
        $ref: "org/APIDOMAIN0001/0.0.1/elementDefs.yaml#/AlternativeID"
  ...      
```

__Updated version:__
```
...
  selection:
    $ref: "#/definitions/Selection"
  ...
  definitions:
  # Selection structures.
    Selection:
      type: object
      description:  Identifies the selection to retrieve information for. Only one of the child elements of this structure are to be provided.
      properties:
        id:
          $ref: "org/APIDOMAIN0001/0.0.2/elementDefs.yaml#/ID"
        card:
          $ref: "org/APIDOMAIN0001/0.0.2/elementDefs.yaml#/AlternativeID"
  ...
```

### Implicit object definitions are to be avoided from the declaration of Body elements when used from collection declarations

An object should be defined in the __*definitions*__ section of the specification or an external document, for __*shared definitions*__, and be referenced from the collection in the Body, instead of the declaration of an implicit object.

The Body should not contain any collection declarations with implicit object definitions; the equivalent of Java inline declarations.

__Ex.: *names element*__:

__Original version:__
```
  ...
  names:
    description: Contact information.
    type: array
    items:
      properties:
        "id":
          $ref: "org/APIDOMAIN0001/0.0.1/elementDefs.yaml#/ID"
        name:
          description: A name can contain up to two lines of name information.
          type: array
          items:
            $ref: "org/APIDOMAIN0001/0.0.1/elementDefs.yaml#/Name"
  ...          
```

__Updated version:__
```
...
  names:
    type: array
    items:
      $ref: "#/definitions/RetrieveNames"
  ...
  definitions:
  RetrieveNames:
    description: Contact information
    type: object
    properties:
      id:
        $ref: "org/APIDOMAIN0001/0.0.2/elementDefs.yaml#/ID"
      name:
        description: A name can contain up to two lines of name information.
        type: array
        items:
          $ref: "org/APIDOMAIN0001/0.0.2/elementDefs.yaml#/Name"
  ...
```



[design first]: /design/design-first/
[light-codegen]: /tool/light-codegen/
[light-rest-4j]: /style/light-rest-4j/
[petstore]: https://github.com/networknt/model-config/tree/master/rest/swagger/petstore/2.0.0
[model-config]: https://github.com/networknt/model-config/tree/master/rest/swagger
[Swagger 2.0 Mock]: /tutorial/generator/swagger-mock/
