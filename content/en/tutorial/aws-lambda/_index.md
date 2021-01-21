---
title: "AWS Lambda Function Tutorial"
date: 2021-01-20T12:46:15-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

There are two different ways to build and deploy Lambda functions described in the [light-aws-lambda][]. The difference is how the cross-cutting concerns are addressed for the Lambda functions. 

* [aws-framework](/tutorial/aws-lambda/lambda-framework/) addresses CCC within the Lambda function built with Java 11 and AWS API Gateway JWT Authorizer.
* [aws-proxy](/tutorial/aws-lambda/lambda-proxy/) addresses CCC in the light-proxy instance so that Lambda function can be built with any language. 

[light-aws-lambda]: /style/light-aws-lambda/
