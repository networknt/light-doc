---
title: "Lambda Invoker"
date: 2021-01-15T14:34:27-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

It is the most important handler for the light-proxy to call Lambda functions deployed to the AWS. By putting this handler at the end of the request/response chain, the light-proxy will address all the cross-cutting concerns before invoking the lambda function mapped to the endpoint. 

### Configuration

The following is the config file for this handler. It will be generated from the OpenAPI specification file with the light-codegen. 

lambda-invoker.yml

```
# The aws region that is used to create the LambdaClient.
region: us-east-1
# The LogType of the execution log of Lambda. Set Tail to include and None to not include.
logType: Tail
# mapping of the endpoints to Lambda functions
functions:
  /pets@get: PetsGetFunction
  /pets@post: PetsPostFunction
  /pets/{petId}@get: PetsPetIdGetFunction
  /pets/{petId}@delete: PetsPetIdDeleteFunction
```

In the above config file, the region defines which region the lambda functions are deployed. 

The logType is set to Tail by default, and it shouldn't be changed. It allows the logs from the lambda function to be tailed in the light-proxy log file. 

The mapping between the endpoints to Lambda functions is generated to allow the lambda-invoker handler to find the target function to call. You can use the full ARN or use the function name only. 


### Request Handler

When the handler is constructed, a LambdaClient object is created. In the handleRequest method, we get method, request path, query parameters, path parameters, headers and body from the exchange. Find out the mapped Lambda function from the path and method combination and invoke the lambda function. 

To ensure that the Lambda function is invoked the same way from AWS API Gateway, we use APIGatewayProxyRequestEvent to invoke the function and expect APIGatewayProxyResponseEvent in return. 


### Security

To allow the light-proxy to invoke Lambda functions, you need to create a AWS role with access to the lambda function. Once you have the role created, you can create a key and secret and put them into the environment variable. 

The following environment variables should be passed to the Docker container. 

* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY

