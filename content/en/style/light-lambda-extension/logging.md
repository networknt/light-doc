---
title: "Lambda Logging"
date: 2020-12-10T10:21:54-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

By default, all logs to the stdout and stderr will be captured by the Lambda runtime and send to CloudWatch. There are two issues with using the CloudWatch for logging. 

Although we can inject the CloudWatch logs to Splunk or Elasticsearch for centralized logging, we need to pay the costs for the CloudWatch. 

Also, the default log doesn't have the TraceabilityId and CorrelationId that is crucial to link the logs for a specific request in a microservices application. 

To work around the above issues, we are going to leverage the [AWS Lambda Logs API](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-logs-api.html). 


To capture and send the logs Splunk and ElasticSearch, an HTTP server needs to be started in the extension layer and leverage the Logs API. The Logs API allows extensions to subscribe to three different logs streams:

* Function logs that the Lambda function generates and writes to stdout or stderr.
* Lambda platform logs, such as the START, END, and REPORT logs.
* Extension logs that extension code generates.

Note the same HTTP server can be used to collect the metrics and send the metrics to the InfluxDB or other services that handles metrics.

