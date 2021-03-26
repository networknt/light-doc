---
title: "Centralized Logging"
date: 2017-11-18T16:29:09-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "service"
    weight: 30
weight: 30	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

Logging is very important for large-scale enterprise applications as it is crucial for the support team to figure out what has happened if something is wrong with the application. Once we move from the monolithic architecture to microservices architecture, logging becomes more complicated as we cannot allow the support team to go to dozens or hundreds VMs or Docker containers to check the log in order to diagnose issues. A centralized logging is a must and some sort of tracing and correlating needs to be built into the log from each service.

When deploying services to a Kubernetes cluster, we need to config our logback.xml to output to the stdout and the log content will be collected by either Logstash or Fluentd depending on the platform. The collected logging info will be ingested to ElasticSearch which can index the logs. Kabana is an UI tool that can be used to view/filter the indexed logs. 

### Correlation ID

Once all the logs are ingested to a central location and indexed together, we need to link all the related logs for a particular request together to assist diagnosis. To correlate all logs for the same request, we have wired in a middleware handler [Correlation][] in the handler chain. The first microservice will be responsible for creating the correlationId and the same correlationId will be propagated to the next service by the client module automatically. For every logging statement, the correlationId will be part of it. To trace a particular transactionId or requestId, search it on the Kabana to find the correlationId and then use the correlationId to filter the logs to find out all services related to the transaction.

### Logging Level

The logback.xml is externalized like other configuration files in the config folder. If you want to change the logging level, you have to update the logback.xml create ConfigMap or Secret then restart the service. 

[Correlation]: /concern/correlation/


 