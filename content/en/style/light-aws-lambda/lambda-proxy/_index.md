---
title: "AWS Lambda Proxy"
date: 2021-01-15T00:33:49-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Light-aws-lambda provides two different ways to address cross-cutting concerns for Lambda function. For enterprise users, it is recommended to use the light-proxy to replace the AWS API Gateway. 

### Lambda Proxy

![lambda-proxy](/images/lambda-proxy.png)


### Light Handler

This is the handler that should be included in the light-proxy as the last handler at the end of the request/response chain. 

For details, please refer to the [lambda-invoker][] handle in the cross-cutting concerns section. 


### Lambda Modules

Although most cross-cutting concerns are addressed in the light-proxy, there is still something that needs to be handled in the lambda function. 


Currently, the only feature that I can think of is the MDC injection for the correlationId, traceabilityId or OpenTracing tracerId. 

To prepare for the future extension, we created [lambda-interceptor][] module that contains an interceptor class to intercept request and response before and after the lambda function is called.


[lambda-invoker]: /style/light-aws-lambda/lambda-proxy/lambda-invoker/
[lambda-interceptor]: /style/light-aws-lambda/lambda-proxy/lambda-interceptor/
