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

Logger-config is a module in the Light-4j framework that will use to get the loggers and their current logging levels. It can also change the logging level for given loggers at runtime (Example: Change logging level to DEBUG for “com.networknt” logger for troubleshooting purpose). The user can also create a brand new logger with a level for debugging issues for a specific package on the target server.

**Note:** This feature is only supposed to be used in a development environment without security enabled. On production, the security must be enabled so that only authorized users can update the logging level through API calls. It is highly recommended to leverage the light-controller(standard) or light-portal(enterprise) to manage all the runtime instances and update them through the UI. 

### Logger-config

There are three Logger Handlers available in logging module.
 
 * LoggerGetHandler : This handler will provide the all available loggers & their current logging levels.
  
     End point    : /logger
   
     Method       : GET
   
     Example URL  : https://localhost:8443/logger
  
 * LoggerGetNameHandler : This handler will provide a logger and its logging level by name.
    
     End point     : /logger/com.networknt
     
     Method        : GET
     
     Example URL   : https://localhost:8443/logger/com.networknt
  
 * LoggerPostHandler : Using this handler we can change the logging level for given
  logger e.g. change logging level to DEBUG for “com.networknt” logger for troubleshooting purpose.
     
      End point     : /logger
      
      Method        : POST
      
      Example Input format : {"name": "com.networknt"level":"DEBUG"}
      
      Example URL    : https://localhost:8443/api/customers/loggers/com.networknt
 
 Input is mandatory, If you provide the input in request body, it will change
 and return the updated logging level for given logger else it will throw an error.
           
 
### Usage

Add dependency.

```xml
<dependency>
  <groupId>com.networknt</groupId>
  <artifactId>logger-config</artifactId>
  <version>${version.light-4j}</version>
</dependency>
```

### Configuration for handler.yml

Add these handlers.

```
- com.networknt.logging.handler.LoggerGetHandler@getLogger
- com.networknt.logging.handler.LoggerGetNameHandler@getNamedLogger
- com.networknt.logging.handler.LoggerPostHandler@postLogger
```

Add the below end points under path.

```
- path: '/logger'
  method: 'get'
  exec:
  # - security
  - getLogger

- path: '/loggers/{loggerName}'
  method: 'get'
  exec:
  # - security
  - getNamedLogger

- path: '/logger'
  method: 'post'
  exec:
  # - security
  - body
  - postLogger
  
```

By default logger is enabled in logger-config.yml. if you want to disable it, just add the logger-config.yml file in your resource folder and make enabled property to false in logger-config.yml.

### Light-Controller

The light-controller is using only the getLogger and postLogger from the UI for each registered instance. For more info, please visit [light-controller](https://doc.networknt.com/service/portal/light-controller/).
