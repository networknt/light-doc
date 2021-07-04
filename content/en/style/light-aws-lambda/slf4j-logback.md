---
title: "Slf4j Logback"
date: 2021-01-15T16:56:51-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This module should be used by the Lambda function regardless of if you are using the lambda-framework or lambda-proxy to use a customized appender for logging. This is to support logback with JSON format and MDC for tracing. The logging statement will be in the same format as the light-proxy so that the accumulated log should be easily parsed by any log aggregation tool like ELK or Splunk. 

When we use the light-code to scaffold the project, this module is included already, and a static logger instance should be created in the handler. 
