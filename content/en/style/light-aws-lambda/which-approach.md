---
title: "Which Approach"
date: 2021-09-12T23:51:48-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

As we have two different approaches to address cross-cutting concerns for the AWS Lambda functions, let's discuss the pros and cons for each approach to guide users on how to choose the best option. 

### Lambda Framework

##### Pros

* Completed Lambda Implementation with shared authorizer deployed on AWS API Gateway.
* Lowest Cost for small and medium organizations when the public cloud is used.
* Tightly integration with AWS products and toolchains, which are familiar to most Lambda developers.

##### Cons

* Only support Java language. Other similar frameworks need to be implemented and maintained if users want to write Lambda functions with .Net, Python, Nodejs etc. 
* Limited due to the Lambda SDK and extension APIs. Hard to integrate with the existing enterprise ecosystem. 
* Vendor lockdown to the AWS cloud solution or ecosystem.


##### Who should use

* Personal and small business focusing on cost reduction with AWS Lambda function pricing model on the public cloud. 
* For Lambda functions that are called in-frequently such batch jobs.
* New startup companies want to simplify their IT solutions. 


### Lambda Proxy


##### Pros

* Support multiple languages for Lamdba functions implementation.
* Same Light-proxy image can be used for Rest APIs and Lambda to address cross-cutting concerns in the same fashion.
* Replace the AWS API Gateway with the capability to extend middleware handlers for easy customization and integration with the existing ecosystem.
* High throughput, low latency with extra functionalities compare with AWS API Gateway.


##### Cons

* Cost of provision proxy instance in ECS with launch type of EC2 or Fargate.

##### Who should use

* For medium and large-size organizations to address cross-cutting concerns for Lambda functions with high throughput.
* For companies with existing EC2 instances pre-allocated.
* For companies that want to customize the security, logging, metrics, auditing, validation etc.
* For companies who are using the light-proxy or http-sidecar already in their cloud solution.
* For organizations who are using Amazon VPC as AWS API Gateway is not free. 

