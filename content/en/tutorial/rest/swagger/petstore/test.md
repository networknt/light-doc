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
reviewed: true
---


The best GUI tool to test RESTful API is [Postman][] and the best command line tool is curl. 

Postman is very easy to set headers and other parameters and save the configuration for 
future usage. If you have a lot of endpoints, it is highly recommended to save all of
them as a project so that these endpoints can be easily retrieved. 

Some people like curl command line and it works as well. In most of the cases, you should
have a file to save the curl commands so that you don't need to reconstruct command each
time. Here is one example to access one of the endpoints petstore serves. 

```
curl -k https://localhost:8443/v2/pet/111
```

Note that -k is to disable the tls certificate verification as we are using a self-signed
certificate on the server. The generated server.yml only enables https by default. 

And the result looks like this. This is the generated example response based on swagger
specification.

```
{
                "photoUrls" : [ "aeiou" ],
                "name" : "doggie",
                "id" : 123456789,
                "category" : {
                  "name" : "aeiou",
                  "id" : 123456789
                },
                "tags" : [
                  {
                    "name" : "aeiou",
                    "id" : 123456789
                  }
                ],
                "status" : "aeiou"
              }
```

Now, let's test the server with an url that is not defined in the specification.

```
curl -k https://localhost:8443/abc
```

And the result is:

```
{"statusCode":404,"code":"ERR10007","message":"INVALID_REQUEST_PATH","description":"Request path cannot be found in the spec."}
```

In fact, the specification is loaded into the service instance by the light-rest-4j 
framework at runtime and there is a module called [Swagger Validator][] that does the 
validation against specification for headers, query parameters, uri parameters and body. 
It also supports validation using JSON schema with an independent library [json-schema-validator][].

Now the server is up and running, let's update generated config files to enable more
features. First let's enable [security][] with OAuth 2.0 JWT token verification as built-in 
[microservices security][] set us apart from other microservices frameworks. 

[json-schema-validator]: https://github.com/networknt/json-schema-validator
[Postman]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en
[Swagger Validator]: /style/light-rest-4j/swagger-validator/
[microservices security]: /architecture/security/
[security]: /tutorial/rest/swagger/petstore/security/
