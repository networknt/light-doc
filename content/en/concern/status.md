---
title: "Error Status"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

## Introduction

When deploying large-scale microservices in an organization, monitoring and alerting are very important for the operation team. The traditional way to monitor monolithic application logs to find potential error status is not feasible anymore. When all service logs are aggregated to the ELK or Splunk, we have a big melting pot of logging statements from different services. To dig useful information from it and identify problematic service instance is not an easy job. It is much simpler to monitor some severe error codes extracted from the logs, and most modern log aggregation tools support that. We need to design and allocate unique error code for each error status across all services in the organization. 


## Features

* Uniqueness across the entire organization. It allows us to identify the component or service with the error code only. 

* Enough context info like message type and description in the dev environment to assist with testing and debugging.

* Remove the message and description in the response on production via configuration to reduce the risk for hackers to figure out how the service works by trying different invalid requests. 

* Log message and description on all environments, including production for support and diagnostic when receiving consumers' calls.

* Support translation between the open-source status.yml error code to an organization-specific error code so that nobody can figure out the meaning from the service response. 

* Customers can provide their implementations to add more context metadata in the status response. 

* For small organizations, each service will have an error code range pre-allocated to ensure uniqueness. 

* For medium or large organizations, light-portal provide error code registration and query with scenarios and resolutions. 

* Each service will have its app-status.yml file to define application-specific error status, and it will be merged to the status.yml provided in the platform during the server start.


## Design

In the scenario that an error happens on the server, a Status object is designed to encapsulate standard HTTP response 4xx and 5xx as well as application-specific error code ERRXXXXX (prefixed with ERR with another five digits) and error message.

Additionally, a description of the error will be available for more info about
the error.

Note that the error code is prefixed with ERR is trying to ensure that there is no false alarm when the monitoring tool processes a stream of logs in searching for a list of significant error code. 

Since we support customized error code per organization, it is good to use your company name acronym for the prefix and translate the light-platform error codes to it. For example, if your company is Example Inc., you can design your error code with EXP00001. The error code's length can be adjusted based on the company size or the number of services expecting to deploy.


#### Data Elements

Here are the four fields in the Status object.
```
    int statusCode;
    String code;
    String message;
    String description;
```

For users who want to add more context info in the development environment, a map of metadata can be added to support some key/value pairs. 

#### Construct the object from status.yml

status.yml is a configuration file that contains all the status error defined
for the application and it has the structure like this.

```
ERR10000:
  statusCode: 401
  code: ERR10000
  message: INVALID_AUTH_TOKEN
  description: Incorrect signature or malformed token in authorization header
ERR10001:
  statusCode: 401
  code: ERR10001
  message: AUTH_TOKEN_EXPIRED
  description: Jwt token in authorization header expired
ERR10005:
  statusCode: 403
  code: ERR10005
  message: AUTH_TOKEN_SCOPE_MISMATCH
  description: Scopes %s in authorization token and specification scopes %s are not matched
  .
  .
  .

}
```

To construct the object from this config

```
    static final String STATUS_METHOD_NOT_ALLOWED = "ERR10008";
    .
    .
    .
    Status status = new Status(STATUS_METHOD_NOT_ALLOWED);

```

To construct the object with arguments to have a description with context information.

```
   return new Status("ERR11000", queryParameter.getName(), swaggerOperation.getPathString().original());
```

Please note that you need to design your description to accept the argument if you want to pass some context info. Take a look at the description of ERR10005, for example. 

#### Convert to JSON response

There are several ways to serialize the object to JSON in response. And string concatenation is almost ten times faster than Jackson ObjectMapper. For one million objects:

```
Jackson Perf 503
ToString Perf 65

```



#### Customize Status JSON format

The default format is a flattened JSON that contains these properties in the status
object. If you want to change the format or adding more fields into it, please refer
to [customize status format][]

#### Error code range allocation
The error code prefixed with ERR with another 5 digits so that it can be easily
scanned in log files. Also, certain error code can be used to trigger an alert
such as email or pager notification on system wide issues.

In order to make sure there is no conflict for error code allocation between
teams, here is the rule

10000-19999 reserved for the framework/system.
   * 10000-10100 security

   * 11000-11999 validation

   * 12000-12999 light-oauth2

20000-29999 common error codes within your business domain.
90000-99999 customize error code that cannot be found in common range.

#### Send the JSON as response

```
    Status status = new Status(STATUS_METHOD_NOT_ALLOWED);
    exchange.setStatusCode(status.getStatusCode());
    exchange.getResponseSender().send(status.toString());
    # rstoreeturn; # return if there are other code below.
```
The above code will terminate the exchange if this is the last line of code in the
handler. Otherwise, you need to add a return statement. 

Alternatively in a handler that implements LightHttpHandler, you can use the following to achieve the same result:
```
    setExchangeStatus(exchange, "ERR10059");
```

#### Stack trace a status
In some cases we need to track where the status has been set when there is not enough logging info provided in the code.
To enable the feature, there are some conditions has to be met:
1. must use the default method **setExchangeStatus(HttpServerExchange exchange, Status status)** which provided by LightHttpHandler  
2. logging level for LightHttpHandler must be set to **TRACE** level

Then whenever a response is sent by setExchangeStatus, the logger will trace the call stack no matter if there is actually an error or not.
To change logging level dynamically, please also see [Light Portal](/getting-started/light-portal)


#### Merging status.yml

The framework has a status.yml defines all the errors within the framework. Once a
company is using this framework, it might have a list of standard error within the
organization. Also, each line of business might have its own error code definitions.
In addition, each individual API/service might have its API specific errors. As you
can see, the errors are in a hierarchical structure and the final one for a particular
API needs to be merged from all levels. 

Three ways can be chosen to merge status:

1. Merging manually and put the final status.yml into your service src/main/resources/config
folder or externalized config folder.

2. Merging automatically by configuring app-status.yml. The contents in app-status.yml are automatically merged with the framework's status.yml when the server starts. However, it should be noted that please do not use the status code that already exists in the status.yml in the app-status.yml, or it will cause an exception.

3. Using [light-config-server][] which can automatically merge status
error codes from multiple levels.


[customize status format]: /faq/customize-status/
[light-config-server]: https://github.com/networknt/light-config-server
