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


The best tool to test RESTful API is [Postman][]. 
It is very easy to set headers and other parameters and save the configuration for 
future usage.

Some people like curl command line and it works as well. Here is one example to access
one of the endpoint petstore serves. 

```
curl -k https://localhost:8443/v1/pets/111
```

Note that -k is to disable the tls certificate verification as we are using a self-signed
certificate on the server. The generated server.yml only enables https by default. 

And the result looks like this. This is the generated example response based on swagger
specification.

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

Now, let's test the server with an url that is not defined in the specification.

```
curl -k https://localhost:8443/abc
```
And the result is:

```
{"statusCode":404,"code":"ERR10007","message":"INVALID_REQUEST_PATH","description":"Request path cannot be found in the spec."}
```

In fact, the specification is loaded into the framework at runtime and there is a
module called Validator that does the validation against specification for headers,
query parameters, uri parameters and body. It also supports validation using JSON
schema with an independent library [json-schema-validator][]


[json-schema-validator]: https://github.com/networknt/json-schema-validator
[Postman]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en
 