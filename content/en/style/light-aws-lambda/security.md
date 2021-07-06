---
title: "Lambda Security"
date: 2020-12-10T08:57:25-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Security is the most important cross-cutting concern that needs to be addressed first for the AWS Lambda Functions. The solution from the AWS is the custom authorizer that is deployed to the API Gateway. For endpoint level authorization, the custom authorizer is not enough. We still need a Lambda extension to verify the scopes against the specification scope definition. 

### Custom Authorizer

The custom authorizer is just a lambda function that is invoked by the API Gateway. It can be built in different languages supported by the Lambda.


There are two different authorizer types based on the input: Token Authorizer and Request Authorizer. 

For the token authorizer, you can only specify one custom header as the token source; however, we need two tokens to capture the original caller and the immediate caller in a microservices application. So our option is the Request Authorizer. 

For a Lambda authorizer of the REQUEST type, API Gateway passes the required request parameters to the authorizer Lambda function as part of the event object. The affected request parameters include headers, path parameters, query string parameters, stage variables, and request context variables. The API caller can set the path parameters, headers, and query string parameters.

As you can see, the authorizer can get almost everything about the request except the request body. Otherwise, we would rather do the validation again the schema in the authorizer. 

There are two different parts of Lambda Security, and the token verification in the authorizer is the first part. In the authorizer, we need to verify the signature of the tokens and ensure they are not expired. The JWT claims will be parsed and enriched the request header so that subsequent Lambda extensions can use. 

The authorizer is shard by multiple Lambda functions, and it can only verify the token is valid in terms of signature and expiration. For endpoint level authorization, we need to perform the scope verification again the OpenAPI specification in the Lambda extensions. 


### Lambda Extension

For each Lambda function, we will package the OpenAPI specification to the extensions for JWT scope verification and schema validation before the handler is called. The scope verification is called first, and the schema validation is called second. The two steps can be implemented in the same extension. 

