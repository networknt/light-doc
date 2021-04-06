---
title: "Request and Response Dump"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This is a handler that dumps the entire request and response into a log file. It should only be used in development mode for debugging purposes as it is very slow.

### Introduction 

This handler can log HTTP request/response info based on configuration.

This is a generic dump handler that may be useful for troubleshooting in a developing/testing environment.

The dump request/response log file should be configured in logback.xml.

The dumping request/response info format can be default format or json format depending on the configuration.

### Configuration
 
The output fields are populated based on the config file dump.yml and here is an example. 

```
# controls how to dump entire request and response to log file. not recommended
# on production if performance is important
enabled: true
# indent size for logging info
indentSize: 4
# use json style or not, if use json, indentSize option will be useless
useJson: false
# if "request: false", disable dumping requests.
request:
  headers: true
  # filter for headers
  filteredHeaders:
  - Postman-Token
  - X-Correlation-Id
  cookies: true
  #filter for cookies
  filteredCookies:
  - Cookie_Gmail
  queryParameters: true
  #filter for queryParameters
  filteredQueryParameters:
  - itemId
  - a
  body: true

response:
  headers: true
  cookies: true
  body: true
  statusCode: true
```
Option explanation:  
request: is the root of request settings  
&nbsp;&nbsp;headers: if true will enable logging headers of REQUEST
&nbsp;&nbsp;filteredHeaders: list of Headers you want to filter out.  
&nbsp;&nbsp;cookies: if true will enable logging cookies of REQUEST  
&nbsp;&nbsp;filteredCookies: list of Cookies you want to filter out.  
&nbsp;&nbsp;queryParameters: if true will enable logging queryParameters of REQUEST  
&nbsp;&nbsp;filteredQueryParameters: list of query parameters you want to filter out.
&nbsp;&nbsp;body: if true will enable logging request body

response: is the root of response settings  
&nbsp;&nbsp;headers: if true will enable logging headers of RESPONSE
&nbsp;&nbsp;filteredHeaders: list of Headers you want to filter out.  
&nbsp;&nbsp;cookies: if true will enable logging cookies of RESPONSE  
&nbsp;&nbsp;filteredCookies: list of Cookies you want to filter out.  
&nbsp;&nbsp;statusCode: if true will enable logging status code of RESPONSE  
&nbsp;&nbsp;body: if true will enable logging response body

### Logback config

The following is an example of the appender defined in the logback.xml or logback-test.xml

```
    <appender name="dump" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>target/dump.log</file> <!-- logfile location -->
        <encoder>
            <pattern>%-5level [%thread] %date{ISO8601} %X{sId} %X{cId} %F:%L - %msg%n
            </pattern> <!-- the layout pattern used to format log entries -->
            <immediateFlush>true</immediateFlush>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>target/test.log.%i.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>5</maxIndex> <!-- max number of archived logs that are kept -->
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>200MB
            </maxFileSize> <!-- The size of the logfile that triggers a switch to a new logfile, and the current one archived -->
        </triggeringPolicy>
    </appender>

```
### Logging example

```
INFO  [XNIO-1 task-3] 2018-12-11 12:20:47,525  7t9TA_UuTNyIQ4BqBVAung DumpHelper.java:23 - Http request/response information:
request:
    cookies:
        Youtube:
            Youtube: abc@youtube
            domain:
            path:
            expires: 
        username:
            username: Balloon
            domain:
            path:
            expires: 
    headers:
        Accept: */*
        cache-control: no-cache
        accept-encoding: gzip, deflate
        User-Agent: PostmanRuntime/7.4.0
        Connection: keep-alive
        Content-Type: application/json
        cookie: Youtube=abc@youtube; Cookie_Gmail=abc@gmail.com; username=Balloon
        content-length: 43
        Host: localhost:8443
    queryParameters:
        aString: 1
        anInteger: "1"
response:
    body: {"content":null,"id":"1"}
    headers:
        Server: L
        Content-Length: 25
        Date: Tue, 11 Dec 2018 17:20:47 GMT
    statusCode: 200
```


### Customized Handler

For some users that need special dump logic or other channels to redirect the dump to, they can create their dump handler and replace the default audit handler in server.yml or handler.yml depending on how you wire your middleware handlers in your application. 
