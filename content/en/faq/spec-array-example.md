---
title: "OpenAPI Specification Array Example"
date: 2018-09-28T20:51:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Most of us know how to create a response example in OpenAPI 3.0 specification in order to allow the light-codegen to generate the example output out of the specification. When dealing with array response, a lot of people don't know how to write the specification to do so. Here is an example and the service is used to test light-router. 

```
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/KeyValue'
              example:
                - key: key1
                  value: value1
                - key: key2
                  value: value2

```

The full specification can be found at https://github.com/networknt/model-config/blob/develop/rest/router/post-service/openapi.yaml

After you generate the project from light-codegen (command line can be found in the README.md in the same folder), you can access the server with the following command. 

```
curl -k -X POST https://localhost:8443/v1/postData -H 'cache-control: no-cache' -H 'content-type: application/json' -H 'host: localhost' -d '[{"key": "key"}, {"key": "key"}]'
```

And the result is 

```
[{"key":"key1","value":"value1"},{"key":"key2","value":"value2"}]
```

