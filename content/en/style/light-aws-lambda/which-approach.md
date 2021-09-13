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

* Completed Lambda Implementation with AWS API Gateway
* Lowest Cost for small and medium organizations


##### Cons

* Only support Java language
* Limited due to the Lambda SDK


##### Who should use

* Personal and small business with focus on the cost reduction. 
* For Lambda functions that are called in-frequently.


### Lambda Proxy


##### Pros

* Support multiple language for Lamdba functions
* Same Light-proxy image can be used for Rest APIs and Lambda to address corss-cutting concerns.
* Replace the AWS API Gateway with capability to extend middleware handlers.
* High throughput


##### Cons

* Cost of provision proxy instance in ECS with launch type of EC2 or Fargate.

##### Who should use

* For medium and large size organization to address cross-cutting concerns for Lambda functions.
* For companies with existing EC2 instances pre-allocated
* For companies want to customize the security, logging, metrics, auditing, valation etc.
* For companies who are using the light-proxy or http-sidecar already.


