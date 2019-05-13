---
title: "JSON Query Parameter"
date: 2019-05-13T15:51:43-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

`style` and `explode` cover the most common serialization methods, but not all. For more complex scenarios (for example, a JSON-formatted object in the query string), you can use the content keyword and specify the media type that defines the serialization format. Or you can use schema to define the JSON object. For more information, please visit [schema vs content][].

In this tutorial, we are going to create a service with one endpoint that accepts a JSON object as a query parameter. The use case is provided by one of our contributors @Dasanko in this [issue](https://github.com/networknt/light-rest-4j/issues/67)

To generate the example project, we need to create a specification file named openapi.yaml and a config.json for the light-codegen. 

You can find these two files and a README.md in the [model-config][] repository. The following is the specification. 

```
openapi: 3.0.0

info:
  title: TestProject
  version: 1.0.0

servers:
  - url: 'https://localhost:8443/api'

paths:
  /testEndPoint:
    get:
      parameters:
        - in: query
          name: TestObject
          schema:
            $ref: '#/components/schemas/TestObject'
      responses:
        '200':
          description: History something something
          content:
            application/json:
              schema:
                type: string

components:
  schemas:
    TestObject:
      type: object
      properties:
        testProperty:
          type: string
```

To generate the project with light-codegen. 

```
cd ~/networknt
rm -rf light-example-4j/rest/openapi/json-query-param
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/json-query-param -m model-config/rest/openapi/json-query-param/openapi.yaml -c model-config/rest/openapi/json-query-param/config.json
```

If you don't want to generate the project yourself, you can check the generated project in the [light-example-4j][]

Now you can start the server in the light-example-4j folder.

```
cd ~/networknt/light-example-4j/rest/openapi/json-query-param
mvn clean install exec:exec
```

Once the server is up and running, you can issue the following commands from another terminal.

```
curl -k -X GET https://localhost:8443/api/testEndPoint?TestObject={testProperty=testing}
curl -k -X GET https://localhost:8443/api/testEndPoint?TestObject=testProperty,testing
curl -k -X GET https://localhost:8443/api/testEndPoint?testProperty=testing
curl -k -X GET https://localhost:8443/api/testEndPoint?TestObject[testProperty]=testing
```

All the above commands should pass the light-rest-4j OpenAPI validator without any error. As there is no business logic implemented in the example application, the response body is empty with a response code of 200. 

If you are interested, you can add the business logic to get the input parameter and return something meaningful. 

[schema vs content]: https://swagger.io/docs/specification/describing-parameters/#schema-vs-content
[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi/json-query-param
[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/rest/openapi/json-query-param