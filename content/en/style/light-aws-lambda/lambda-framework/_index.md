---
title: "AWS Lambda Framework"
date: 2021-01-15T00:33:42-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

### Lambda Framework

![lambda-framework](/images/lambda-framework.png)

### AWS Lambda layers

In the past, the only way we can add cross-cutting concerns to the Lambda function is through lambda layers. 

https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html


You can configure your Lambda function to pull in additional code and content in the form of layers. A layer is a .zip file archive that contains libraries, a custom runtime, or other dependencies. You can use libraries in your function with layers without needing to include them in your deployment package.

A function can use up to 5 layers at a time. The total unzipped size of the function and all layers can't exceed the unzipped deployment package size limit of 250 MB. 

### Custom Authorizer

Some implemented layers of token authorizer integrated with AWS API Gateway are available for Lambda functions; however, unless we can customize them to capture the claims and pass them to the Lambda functions, there is no way for subsequent logging, auditing, tracing and metrics handling. 

We can create a custom authorizer using the light-4j VerifyHelper to verify the token signature and then extract the claim like client_id and user_id for other subsequent extensions on the AWS API Gateway.

The module is [jwt-authorizer][] which is a Lambda function that is deployed with the AWS API Gateway.


### AWS Lambda Extensions


Extensions are a new way for tools to integrate deeply into the Lambda environment. There is no complicated installation or configuration, and this simplified experience makes it easier for you to use your preferred tools across your application portfolio today.

Extensions are built using the new Lambda Extensions API, which provides a way for tools to get greater control during function initialization, invocation, and shut down. This API builds on the existing Lambda Runtime API, enabling you to bring custom runtimes to Lambda.

You deploy extensions as Lambda layers, which are ZIP archives containing shared libraries or other dependencies.

Extensions can run in either of two modes, internal and external.

Internal extensions run as part of the runtime process, in-process with your code. They are not separate processes. Internal extensions allow you to modify the runtime process startup using language-specific environment variables and wrapper scripts. 

External extensions allow you to run separate processes from the runtime but still within the same execution environment as the Lambda function. External extensions can start before the runtime process and can continue after the runtime shuts down. 

External extensions can be written in a different language to the function.

Although external extensions can be written separately, the functionality is very limited. They depend on the Lambda extension API and can only intercept Init, Invoke and Shutdown events without the request detail. The API's purpose is to allow users to hook up other metrics collection tools to allow third-party products to replace CloudWatch. 

### Middleware Handlers

Middleware handlers are interceptors that are injected in the request and response chain to address some cross-cutting concerns. They are wired in the [request-handler][] that will call the business logic.

##### JWT Scope Verification

The [scope-verifier][] is the first middleware handler called by the request-handler to verify the scopes in the JWT tokens against the OpenAPI specification. 

##### Validation

With a specification or JSON schema included in the deployment, we can use the OpenAPI validation or schema validation extension called before forward the request to the Lambda function. 

##### Logging

In the eadp-light-proxy, we have a handler to get the correlationId from the HTTP header and put it into the MDC of the logback to ensure that all logging entries have the correlationId. We can create an internal extension for the same purpose but need to get familiar with the logging API.  

Further, we can capture the logs with the extension and send them to Splunk to avoid parsing the log from the CloudWatch. It can save costs significantly when using Lambda functions. 

##### Metrics, Auditing and Tracing

To enable integration with other tools for telemetry data collection is the primary purpose to provide the extensions API for AWS. We can create extensions that monitor the invocation and capture telemetry data just like other microservices. 



https://aws.amazon.com/blogs/compute/introducing-aws-lambda-extensions-in-preview/
https://aws.amazon.com/about-aws/whats-new/2020/10/aws-lambda-extensions-integrate-operational-tools/
https://aws.amazon.com/blogs/compute/building-extensions-for-aws-lambda-in-preview/
https://github.com/aws-samples/aws-lambda-extensions/

### AWS Lambda Container Image

The most important roadblock for JVM language to create Lambda functions is the code start. It takes more time to start a JVM application and warm-up for the optimization to take effect. It is against the nature of the Lambda function that might only be invoked once after a time. 

To resolve this code start issue, we can create a customized container image with GraalVM to have the AOT (Ahead of Time) compilation for our Lamba functions and extensions. 

Further, we can have the Lamba extensions build with the container image regardless of their language for the Lambda functions. 


https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/
https://aws.amazon.com/about-aws/whats-new/2020/12/aws-lambda-now-supports-container-images-as-a-packaging-format/
https://www.graalvm.org/



### Project Generator

Once we completed the above steps one and two, we will create a generator to scaffold a Lambda function project with all the extensions built-in. Also, we will generate the SAM templete.yml with schema definitions to validate the request at runtime. We can also add OpenAPI specifications to the template for validation if we want to ensure the standard within the organization. The second link below shows us how to use the OpenAPI specification. 

https://github.com/aws-samples/aws-sam-java-rest
https://github.com/jvillane/aws-sam-step-functions-lambda

[jwt-authorizer]: /style/light-aws-lambda/lambda-framework/jwt-authorizer/
[scope-verifier]: /style/light-aws-lambda/lambda-framework/scope-verifier/
[request-handler]: /style/light-aws-lambda/lambda-framework/request-handler/
