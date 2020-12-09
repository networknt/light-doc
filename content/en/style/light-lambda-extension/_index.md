---
title: "Light Lambda Extension"
date: 2020-12-08T16:48:56-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In the light-platform, we are using the middleware handlers to address cross-cutting concerns for microservices in the request/response chain in the same service process or within a light-proxy/router at the network level. 

With more and more serverless Lambda functions deployed to the AWS cloud, we need to find a way to address the cross-cutting concerns in the same fashion as the light-4j so that the Lambda functions can be integrated with the light ecosystem for security, logging, tracing, auditing and metrics. 


There are different options to implement cross-cutting concerns for Lambda functions and they are all built one upon another.


### AWS Lambda layers

In the past, the only way we can add cross-cutting concerns to the Lambda function is through lambda layers. 

https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html

You can configure your Lambda function to pull in additional code and content in the form of layers. A layer is a .zip file archive that contains libraries, a custom runtime, or other dependencies. With layers, you can use libraries in your function without needing to include them in your deployment package.

A function can use up to 5 layers at a time. The total unzipped size of the function and all layers can't exceed the unzipped deployment package size limit of 250 MB. 


### AWS Lambda Extensions

Extensions are a new way for tools to integrate deeply into the Lambda environment. There is no complicated installation or configuration, and this simplified experience makes it easier for you to use your preferred tools across your application portfolio today.

Extensions are built using the new Lambda Extensions API, which provides a way for tools to get greater control during function initialization, invocation, and shut down. This API builds on the existing Lambda Runtime API, which enables you to bring custom runtimes to Lambda.

You deploy extensions as Lambda layers, which are ZIP archives containing shared libraries or other dependencies.

Extensions can run in either of two modes, internal and external.

Internal extensions run as part of the runtime process, in-process with your code. They are not separate processes. Internal extensions allow you to modify the startup of the runtime process using language-specific environment variables and wrapper scripts. 

External extensions allow you to run separate processes from the runtime but still within the same execution environment as the Lambda function. External extensions can start before the runtime process, and can continue after the runtime shuts down. 

External extensions can be written in a different language to the function.

##### Security

Some implemented layers of token authorizer are available for Lambda functions; however unless we can customize them to capture the claims for subsequent logging, auditing, tracing and metrics. 

We can create a Lambda extension using the light-4j VerifyHelper to verify the token and then extract the claim like client_id and user_id for other subsequent extensions. 


##### Logging

In the eadp-light-proxy, we have a handler to get the correlationId from the HTTP header and put it into the MDC of the logback to ensure that all logging entries have the correlationId. We can create an internal extension for the same purpose but need to get familiar with the logging API.  

Further, we can capture the logs with the extension and send them to Splunk to avoid parsing the log from the CloudWatch. It can save costs significantly when using Lambda functions. 

##### Metrics, Auditing and Tracing

To enable integration with other tools for telemetry data collection is the primary purpose to provide the extensions API for AWS. We can create extensions that monitor the invocation and capture telemetry data just like other microservices. 

##### Validation

With a specification or JSON schema included in the deployment, we can use the OpenAPI validation or schema validation extension called before forward the request to the Lambda function. 


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

