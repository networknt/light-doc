---
title: "Request Response Transformation"
date: 2022-07-20T23:33:44-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this tutorial, we will demo the request and response transformation based on the rule engine in the light-gateway. The same handlers/interceptors can be used in the http-sidecar and any application built with the light-4j. 

The demo will be based on three real use cases from our customers. We will show you the configuration and run the test with the petstore backend API to show you how the light-gateway works on the request and response chain. 


The backend petstore API can be found [here](https://github.com/networknt/light-example-4j/tree/master/rest/petstore-maven-single)

The transformation rule implementations can found [here](https://github.com/networknt/light-example-4j/tree/master/rule)

The light-gateway configuration for the demo can be found [here](https://github.com/networknt/light-gateway/tree/master/config/client-proxy-transform)


The light-example-4j default branch is the release; Here we are using the master branch, which is in sync with light-gateway master branch. 

### Prepare Environment

Follow the steps below: 

You need to checkout the light-gateway and light-example-4j both master branch. 

* Build the light-example-4j/rest/petstore-maven-single

* Build three modules under light-example-4j/rule folder. 

* Copy the rule jar files to light-gateway/rulesjar folder. 

* Start the petstore from command line. 

* Start the light-gateway from the IDE and add the rulesjar folder to the classpath.


{{< youtube VwlCKeIS7Uo >}}


### Request Path Update from a Header

One of the project teams is migrating from SAG gateway to light-gateway, and they don't want to update any of the consumers. The consumer's request path needs to be rewritten to add a prefix from the HTTP header named context. Although the light-gateway router.yml gives you the capability to rewrite the request path based on the incoming request path, it doesn't support rewriting the request path from the header values. To do that, we will leverage the interceptor RequestTransformerInterceptor


{{< youtube xZ_0XZRZHVs >}}


### Response Structure Transform

One of the customers has defined a standard response structure with data and a list of notifications for error messages. When they upgrade all the services to this standard response, all existing native consumers are broken as they are expecting another format of the response. To avoid changing the native consumer, we have added a client proxy instance to the native consumer to transform the notifications response to something the native consumer is expecting. 

{{< youtube vx9NOQtb0QY >}}


### REST to SOAP Transform

A team maintains a SOAP service and wants to rewrite it with REST. They have completed an open API specification but haven't implemented it yet. A new consumer is onboarding to build a brand new application that wants to leverage the REST API instead of SOAP API. To support that, we put a light-gateway instance in between to translate the REST JSON request to SOAP XML request and SOAP XML response to REST JSON response. 


{{< youtube V4WYUGqbFaM >}}

### SOAP Security Transform

One of our customers is accessing a Cannex Soap API from the corporate network with light-gateway via the generic [ExternalServiceHandler](/concern/external-handler/). For all the consumers, they don't need to set the security header in the SOAP request; the light-gateway intercepts the request body and adds security header with the correct credentials to access the Cannex. This is very similar to the above REST to SOAP request transformer; however, it is a little bit complicated as we are accessing an external service instead of internal services. For detailed configuration, please review this [PR](https://github.com/networknt/light-gateway/pull/69)

{{< youtube MTA6Pf1TjaU >}}
