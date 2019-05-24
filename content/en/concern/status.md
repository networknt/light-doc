---
title: "Status"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
---

In the scenario that error happens on the server, a Status object is designed
to encapsulate standard http response 4xx and 5xx as well as application specific
error code ERRXXXXX (prefixed with ERR with another 5 digits) and error message.
Additionally, an description of the error will be available for more info about
the error.

Note that the error code is prefixed with ERR is trying to ensure that there is
no false alarm when monitoring tool process a stream of logs in searching for a
list of important error code. 

# Data Elements

Here are the four fields in the Status object.
```
    int statusCode;
    String code;
    String message;
    String description;
```

# Construct the object from status.yml
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

# Convert to JSON response

There are several way to serialize the object to JSON in response. And string
concat is almost 10 times faster than Jackson ObjectMapper. For one million
objects:

```
Jackson Perf 503
ToString Perf 65

```

# Customize Status JSON format

The default format is a flattened JSON that contains these properties in the status
object. If you want to change the format or adding more fields into it, please refer
to [customize status format][]

# Error code range allocation
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

# Send the JSON as response

```
    Status status = new Status(STATUS_METHOD_NOT_ALLOWED);
    exchange.setStatusCode(status.getStatusCode());
    exchange.getResponseSender().send(status.toString());
    # rstoreeturn; # return if there are other code below.
```

The above code will terminate the exchange if this is the last line of code in the
handler. Otherwise, you need to add a return statement. 

# Merging status.yml

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
