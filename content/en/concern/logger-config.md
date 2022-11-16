---
title: "Logger Config"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Logger-config is a module in the Light-4j framework that will be used to get the loggers and their current logging levels. It can also change the logging level for given loggers at runtime (Example: Change logging level to DEBUG for “com.networknt” logger for troubleshooting purposes). The user can also create a brand new logger with a level for debugging issues for a specific package or class on the target server.

**Note:** This feature is only supposed to be used in a development environment without security enabled. On production, the security must be enabled so only authorized users can update the logging level through API calls. It is highly recommended to leverage the light controller (standard) or light portal (enterprise) to manage all the runtime instances and update them through the UI.

### Specification

The specification is injected to any light-rest-4j application via [openapi-inject.yml](https://github.com/networknt/light-rest-4j/blob/master/openapi-meta/src/main/resources/config/openapi-inject.yml) when the application specific openapi.yaml file is loaded. 

Here are the endpoint definitions.

```
  /adm/logger:
    get:
      description: to get the current logging settings
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin
    post:
      description: to modify the logging settings
      requestBody:
        description: to update logging settings
        content:
          application/json:
            schema:
              type: "array"
              items:
                type: "object"
                properties:
                  name:
                    type: "string"
                  level:
                    type: "string"
        required: true
      security:
        - admin-scope:
            # okta doesn't support either for now
            # for control pane to access all businesses' admin endpoints
            - admin
            # for each business to access their own admin endpoints
            - ${server.serviceId}/admin

  /adm/logger/content:
    get:
      description: to get the content of the logs from this service.
      security:
        - admin-scope:
            - admin
            - ${server.serviceId}/admin

```

The first endpoint /adm/logger@get is used to get all the loggers configured in the logback.xml on the target server. 

The second endpoint /adm/logger@post is used to update existing loggers or add new loggers. The body of the request is an array of maps in JSON format. Each map will have the key as the logger name and the value as the logging level.

The third endpoint /adm/logger/content@get is used to get the logs from the server. It is used by the control pane and the light portal UI to display the logs from the target server.

### Configuration

If you use light-codegen to generate a light-rest-4j application, the handler.yml will have this endpoint configured. Also, you can find the examples in http-sidecar and light-gateway repositories. 

handler.yml

```
paths:
  .
  .
  .
  - path: '/adm/logger'
    method: 'get'
    exec:
      - admin
      - getLogger

  - path: '/adm/logger'
    method: 'post'
    exec:
      - admin
      - postLogger

  - path: '/adm/logger/content'
    method: 'GET'
    exec:
      - admin
      - getLogContents
   .
   .
   .

```

If the handler.chains.default is externalized, you need to register the handler classes and give aliases to them in the values.yml file.

values.yml

```
handler.handlers:
  .
  .
  .
  - com.networknt.logging.handler.LoggerGetHandler@getLogger
  - com.networknt.logging.handler.LoggerPostHandler@postLogger
  - com.networknt.logging.handler.LoggerGetLogContentsHandler@getLogContents
```

The logger-config module has its configuration to control if the module is enabled or not. 

Here is the default logging.yml

```
---
# Logging endpoint that can output the logger with logging levels.

# Indicate if the logging info is enable or not.
enabled: ${logging.enabled:true}
# Indicate the default time period backward in millisecond for log content retrieve. Default is  hour which indicate system will retrieve one hour log by default
logStart: ${logging.logStart:600000}
```

You can disable the logger-config module and change the log retrieval timestamp in the values.yml file to overwrite the default values. 


### Security

Depending on how the security is configured on the server, you need to get a token and put it into the Authorization header to access the logger endpoints. If the light portal is used, the request can be sent from the control pane to each individual service instance. When a user logs in to the light portal, a token will be created with the proper scope to access the admin endpoints. 

You can disable the security in the values.yml file for a simple local test. 

values.yml

```
# openapi-security.yml
openapi-security.enableVerifyJwt: false
```

### Dependency

Add dependency in the application pom.xml file.

```xml
<dependency>
  <groupId>com.networknt</groupId>
  <artifactId>logger-config</artifactId>
  <version>${version.light-4j}</version>
</dependency>
```

### Usage

By default, the logger-config module is enabled in logging.yml. If you want to disable it, just add an entry "logging.enabled: false" in values.yml to overwrite the default value to false. 

To get the loggers and their levels. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger'
```

And the result should be something like. 

```
[
    {
        "name": "ROOT",
        "level": "ERROR"
    },
    {
        "name": "Audit",
        "level": "INFO"
    },
    {
        "name": "com.networknt",
        "level": "TRACE"
    }
]
```

To change the loging level for com.networknt to ERROR and add another logger "com.networknt.server.Server" to TRACE.

```
curl -k --location --request POST 'https://localhost:8443/adm/logger' \
--header 'Content-Type: application/json' \
--data-raw '[
    {
        "name": "com.networknt",
        "level": "ERROR"
    },
    {
        "name": "com.networknt.handler.ProxyHandler",
        "level": "TRACE"
    }
]'
```
And the result is.

```
[
    {
        "name": "com.networknt",
        "level": "ERROR"
    },
    {
        "name": "com.networknt.handler.ProxyHandler",
        "level": "TRACE"
    }
]
```

On the server, the trace should be enabled for the com.networknt.server.Server class and we can get the logs with the following default command. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content'
```

The result should be something like below. 

```
{
    "ROOT": {
        "total": 1,
        "logs": [
            {
                "timestamp": "2022-11-11T14:58:21.880-0500",
                "thread": "XNIO-1 task-1",
                "level": "ERROR",
                "logger": "com.networknt.handler.LightHttpHandler",
                "correlationId": "toseSd8CTg2a1aECCYKkqA",
                "serviceId": "",
                "class": "LightHttpHandler.java",
                "lineNumber": "com.networknt.handler.LightHttpHandler:116",
                "method": "setExchangeStatus",
                "logMessage": {
                    "statusCode": 401,
                    "code": "ERR10002",
                    "message": "MISSING_AUTH_TOKEN",
                    "description": "No Authorization header or the token is not bearer type",
                    "severity": "ERROR"
                }
            }
        ]
    },
    "com.networknt": {
        "total": 1,
        "logs": [
            {
                "timestamp": "2022-11-11T14:58:21.880-0500",
                "thread": "XNIO-1 task-1",
                "level": "ERROR",
                "logger": "com.networknt.handler.LightHttpHandler",
                "correlationId": "toseSd8CTg2a1aECCYKkqA",
                "serviceId": "",
                "class": "LightHttpHandler.java",
                "lineNumber": "com.networknt.handler.LightHttpHandler:116",
                "method": "setExchangeStatus",
                "logMessage": {
                    "statusCode": 401,
                    "code": "ERR10002",
                    "message": "MISSING_AUTH_TOKEN",
                    "description": "No Authorization header or the token is not bearer type",
                    "severity": "ERROR"
                }
            }
        ]
    }
}
```

The default logging level is ERROR and we can change it to other levels in the query parameter. Also, we can choose the logger we need to get the output. The following commmand will only get logs for logger com.networknt and the level is trace. 


```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE'
```

The result should be something like below. 

```
{
    "com.networknt": {
        "total": 92,
        "logs": [
            {
                "timestamp": "2022-11-11T17:28:28.829-0500",
                "thread": "XNIO-1 task-1",
                "level": "DEBUG",
                "logger": "com.networknt.limit.RateLimiter",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RateLimiter.java",
                "lineNumber": "com.networknt.limit.RateLimiter:250",
                "method": "isAllowByServer",
                "logMessage": "CurrentTimeWindow:1668205708 Result:true  Count:0"
            },
            {
                "timestamp": "2022-11-11T17:28:28.830-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.openapi.JwtVerifyHandler",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "JwtVerifyHandler.java",
                "lineNumber": "com.networknt.openapi.JwtVerifyHandler:96",
                "method": "handleRequest",
                "logMessage": "Skip request path base on skipPathPrefixes for /v1/pets"
            },
            {
                "timestamp": "2022-11-11T17:28:28.831-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.reqtrans.RequestTransformerInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestTransformerInterceptor.java",
                "lineNumber": "com.networknt.reqtrans.RequestTransformerInterceptor:77",
                "method": "handleRequest",
                "logMessage": "RequestTransformerHandler.handleRequest is called."
            },
            {
                "timestamp": "2022-11-11T17:28:28.833-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.body.RequestBodyInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestBodyInterceptor.java",
                "lineNumber": "com.networknt.body.RequestBodyInterceptor:73",
                "method": "handleRequest",
                "logMessage": "RequestBodyInterceptor is called."
            },
            {
                "timestamp": "2022-11-11T17:28:28.834-0500",
                "thread": "XNIO-1 task-1",
                "level": "TRACE",
                "logger": "com.networknt.body.RequestBodyInterceptor",
                "correlationId": "sSWYGOqbTzq8zpS5JQJCAA",
                "serviceId": "",
                "class": "RequestBodyInterceptor.java",
                "lineNumber": "com.networknt.body.RequestBodyInterceptor:119",
                "method": "shouldAttachBody",
                "logMessage": "hasBody = false isPathConfigured = true"
            },
            .
            .
            .

```

You can change the pagination limit and offset from default 100 and 0. 

For example. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE&limit=10&offset%20=10'
```

Above command will get 10 log statements starting from line 10. You can specify startTime and endTime in epoch milliseconds to get logs for a specific period. 

```
curl -k --location --request GET 'https://localhost:8443/adm/logger/content?loggerName=com.networknt&loggerLevel=TRACE&startTime=1668000000000&endTime=1668206563000'
```


### Light-Controller

The light-controller is using only the getLogger and postLogger from the UI for each registered instance. For more info, please visit [light-controller](https://doc.networknt.com/service/controller/).


{{< youtube 5iHRD8sRKY0 >}}
