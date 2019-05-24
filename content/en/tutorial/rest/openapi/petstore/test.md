---
title: "Test"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 60
aliases: []
toc: false
draft: false
---


The best GUI tool to test a RESTful API is [Postman][] and the best command line tool is curl. 

Postman makes it very easy to set headers and query parameters while being able to save the configuration 
for future usage. If you have a lot of endpoints, it is highly recommended to save all of
them as a project so that these endpoints can be easily retrieved. 

Some people like the curl command line and it works as well. In most of the cases, you should
save the curl commands to a file which saves you time when reconstructing each command. Here is 
one example to access one of the endpoints the petstore API serves. 

```
curl -k https://localhost:8443/v1/pets/111
```

Note that -k is to disable the TLS certificate verification as we are using a self-signed
certificate on the server. The generated server.yml enables HTTPS only by default. 

The response looks like this, which is the generated example response based on the Swagger
specification.

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

Now, let's test the server with a url that is not defined in the specification.

```
curl -k https://localhost:8443/abc
```

And the response is:

```
{"statusCode":400,"code":"ERR10016","message":"MISSING_HANDlER","description":"Could not find the handler for the endpoint Method: GET, RequestPath: /abc.","severity":"ERROR"}
```

The specification is loaded into the framework at runtime where there is a
module called OpenAPI Validator that does the validation against the specification for headers,
query parameters, URI parameters and body. It also supports validation using JSON
schema with an independent library [json-schema-validator][].

Now the server is up and running, let's update the generated config files to enable more
features. First let's enable [security][] with OAuth 2.0 JWT token verification which is built into
[microservices security][] which sets us apart from other microservices frameworks. 

[json-schema-validator]: https://github.com/networknt/json-schema-validator
[Postman]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en
[OpenAPI Validator]: /style/light-rest-4j/openapi-validator/
[microservices security]: /architecture/security/
[security]: /tutorial/rest/openapi/petstore/security/
