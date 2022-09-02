---
title: "Audit Log"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

A built-in audit handler writes logs into audit.log file, which is set up in the logback appender. The end user can add more customized audit handlers if needed. For example, some customers write audit info into a relational database, while others write it into a Kafka topic.

In the audit module, there is AuditHandler which is generic and configurable with the audit.yml config file. 

There is another audit provided by the light-4j framework called DumpHandler in the [dump](concern/dump/) module.

### Introduction 

This handler logs several fields from the request headers, query parameters, path parameters, request body, response status, response time and response body. All the fields are configurable in the audit.yml file. 

This is a generic audit handler that dumps the most important info per request basis. The following elements will be logged if they are available in the AuditInfo object attached to the exchange. This object will be populated by other upstream handlers like swagger-meta and swagger-security for the light-rest-4j, light-graphql-4j and light-hybrid-4j frameworks.

This handler can be used on production, but be aware that it will impact overall performance. Turning off statusCode and responseTime can make it faster, as these have to be captured on the response chain instead of the request chain. Also, if the request body and response body are captured, it will be much slower. 

For most businesses and the majority of microservices, you don't need to enable this handler due to performance reasons. There are situations during troubleshooting or integration testing where teams need to collect more information based on the success or failure of the execution or collect complete request and response payloads. Teams need to recognize that collecting this information will negatively impact performance, and it is not recommended to use it in production environments.

When the light-portal is used to manage all the service instances, we can reload the configuration if needed from the light-config-server. This will allow us to enable the audit on demand. 

The default audit log will be audit.log config in the default logback.xml; however, it can be changed to syslog or Kafka with a customized appender.

The majority of the fields in the audit log are collected in request and response; however, to allow users to customize it, we have put an attachment into the exchange to allow other handlers to write important info into it. The audit.yml can control which fields should be included in the final log.

By default, the following fields are included:

 * statusCode
 * responseTime
 * X-Correlation-Id (from request headers)
 * X-Traceability-Id (if available from request headers)
 * caller_id (if available from request headers)
 * client_id (from the jwt token)
 * user_id (from the jwt token if available)
 * scope_client_id (available if called by another API)
 * endpoint (uriPattern@method)
 * serviceId (from server.yml)
 * requestBody (the request body payload from RequestBodyInterceptor)
 * responseBody (the response payload from ResponseBodyInterceptor)

The audit.log is in JSON format, and it is easy to be parsed and monitored. 

### Configuration

You will need to define com.networknt.audit.AuditHandler@audit in handler.yml then add this “audit” handler to the handler chain.
 
The output fields are populated based on the config file audit.yml. Here is an example:

```
# AuditHandler will pick some important fields from headers and tokens and logs into an audit appender
# defined in the logback.xml configuration file.
---
# Enable Audit Logging
enabled: ${audit.enabled:true}

# Enable mask in the audit log
mask: ${audit.mask:true}

# Output response status code
statusCode: ${audit.statusCode:true}

# Output response time
responseTime: ${audit.responseTime:true}

# when auditOnError is true:
#  - it will only log when status code >= 400
# when auditOnError is false:
#  - it will log on every request
# log level is controlled by logLevel
auditOnError: ${audit.auditOnError:false}

# log level; by default it is set to info. If you want to change it to error, set to true.
logLevelIsError: ${audit.logLevelIsError:false}

# the format for outputting the timestamp, if the format is not specified or invalid, will use a long value.
# for some users that will process the audit log manually, you can use yyyy-MM-dd'T'HH:mm:ss.SSSZ as format.
timestampFormat: ${audit.timestampFormat:}

# Output header elements. You can add more if you want. If multiple values, you can use a comma separated
# string as default value in the template and values.yml. You can also use a list of strings in YAML format.
headers: ${audit.headers:X-Correlation-Id, X-Traceability-Id,caller_id}

# Correlation Id
# - X-Correlation-Id
# Traceability Id
# - X-Traceability-Id
# caller id for metrics
# - caller_id

# Output from id token and access token. If multiple values, you can use a comma separated string as default
# value in the template and values.yml. You can also use a list of strings in YAML format.
audit: ${audit.audit:client_id, user_id, scope_client_id, endpoint, serviceId, requestBody, responseBody}

# Client Id
# - client_id
# User Id in id token, this is optional
# - user_id
# Client Id in scope/access token, this is optional
# - scope_client_id
# Request endpoint uri@method.
# - endpoint
# Service ID assigned to the service, this is optional and must be set by the service in its implementation
# - serviceId
# Request Body, this is optional and must be set by the service in its implementation
# - requestBody
# Response payload, this is optional and must be set by the service in its implementation
# - responseBody

```

The following values.yml sections in the light-gateway/client-proxy-transform folder demonstrated the capture of the request and response body. 

values.yml

```
# service.yml
service.singletons:
  .
  .
  .
  - com.networknt.handler.ResponseInterceptor:
      - com.networknt.body.ResponseBodyInterceptor
      - com.networknt.restrans.ResponseTransformerInterceptor
  - com.networknt.handler.RequestInterceptor:
      - com.networknt.body.RequestBodyInterceptor
      - com.networknt.reqtrans.RequestTransformerInterceptor


# handler.yml
handler.handlers:
  .
  .
  .
  - com.networknt.handler.ResponseInterceptorInjectionHandler@responseInterceptor
  - com.networknt.handler.RequestInterceptorInjectionHandler@requestInterceptor

handler.chains.default:
  - exception
  - traceability
  - correlation
  - requestInterceptor
  - responseInterceptor
  - header
  .
  .
  .

```



### Logback config

The following is the appender defined in the logback.xml or logback-test.xml

```
    <appender name="audit" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/audit.log.json</file> <!-- logfile location -->
        <encoder>
            <pattern>%msg%n</pattern> <!-- the layout pattern used to format log entries -->
            <immediateFlush>true</immediateFlush>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>log/audit.log.json.%i.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>5</maxIndex> <!-- max number of archived logs that are kept -->
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>200MB
            </maxFileSize> <!-- The size of the logfile that triggers a switch to a new logfile, and the current one archived -->
        </triggeringPolicy>
    </appender>

    <logger name="audit" level="info" additivity="false">
        <appender-ref ref="audit"/>
    </logger>

```

The above configuration is for a normal text parser. If you want the audit.log to format the message in a JSON object, you can consider a customized JSON encoder. 

### Logging example

Below is the example with default configs:

```
2022-08-04 13:45:25,278 {"timestamp":1659635113471,"X-Correlation-Id":"wGppOjjURkK-M4n7J4XUXQ","X-Traceability-Id":null,"caller_id":null,"serviceId":"com.networknt.client-gateway-1.0.0","statusCode":404,"responseTime":11807,"client_id":null,"user_id":null,"scope_client_id":null,"endpoint":"/pets@get","requestBody":null,"responseBody":null}
```

With different timestamp format:
```
2022-08-05 11:00:07,950 {"timestamp":"2022-08-05T11:00:07.684-0400","X-Correlation-Id":"8nA9wnVSTreolXNrrH8E4g","X-Traceability-Id":null,"caller_id":null,"serviceId":"com.networknt.client-gateway-1.0.0","statusCode":404,"responseTime":247,"client_id":null,"user_id":null,"scope_client_id":null,"endpoint":"/pets@get","requestBody":null,"responseBody":"{\"statusCode\":404,\"code\":\"ERR10048\",\"message\":\"PATH_NOT_IMPLEMENTED\",\"description\":\"No handler defined for path '/pets'\",\"severity\":\"ERROR\"}"}

```

With request body and response body.

```
2022-08-04 14:42:50,142 {"timestamp":1659638569905,"X-Correlation-Id":"cZ_izfmDSxW1RiDlGD5piw","X-Traceability-Id":null,"caller_id":null,"requestBody":"{\"name\":\"Poppy\",\"color\":\"RED\",\"petals\":9}","serviceId":"com.networknt.client-gateway-1.0.0","statusCode":400,"responseTime":237,"client_id":null,"user_id":null,"scope_client_id":null,"endpoint":"/v1/flowers@post","responseBody":"{\"statusCode\":400,\"code\":\"ERR10015\",\"message\":\"CONTENT_TYPE_MISMATCH\",\"description\":\"Content type in header application/json doesn't match the body.\",\"severity\":\"ERROR\"}"}

```

### Customized Handler

For some users that need special audit logic or other channels to redirect the audit to, they can create their audit handler and replace the default audit handler in handler.yml, depending on how you wire your middleware handlers in your application. 

