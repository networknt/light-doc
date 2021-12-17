---
title: "Log Masking"
date: 2021-12-14T23:30:36-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Most log masking modules can only handle the masking with Regex, and it is not enough in a complicated JSON object logging. We need to define the Regex with JSONPath to identify the masking data. 

Light-4j mask module supports String, Regex and JSON masking with a configuration file mask.yml customized for a particular API. To leverage that, we can allow the backend API to send the logs to the http-sidecar for masking before ingesting them to the target indexing system.

Most logging frameworks support different types of appenders so that the logs can be piped to another destination like a filesystem, a database and another server. In Java, the logback has a SocketAppender to send the logs to the sidecar proxy. 

For other languages, we need to find a logging framework that supports remote HTTP logging, and this is the common feature for most logging frameworks today in the cloud environment. 

### Kafka vs Splunk Connect

By push the logging statements to a Kafka topic, there are pros and cons. 

Pros: 

* Remove the effort for each API to onboard Splunk. A Kafka Splunk connector will handler the log ingestion for all APIs.
* Reduce the effort to move to other logging index systems like open-source ELK. 
* Easy to attach microservices to process the logs with Kafka Streams. For example, Alert, Run Boot Automation etc.
* We can pipe the logs to other destinations for further analysis to identify behaviour changes for APIs. For example, AI Log Analysis.
* Simplify the overlay configuration for the API as we don't write log files to a certain directory. 
* Eliminate the Splunk Connect deployed to each Kubernetes nodes. 

Cons:

* Depending on the High-Available Kafka cluster.
* Extra components with increased maintenance effort.

Ingestion to Splunk with Splunk Connect for Kubernetes.

Pros: 

* The same vendor provides all components for easy support.

Cons:

* Tightly coupled with one provider with Vendor lock-in.
* Hard to onboard new API projects as the process is very complicated and takes a long time.


### Log on Demand

Splunk is charging based on the volume, and Kafka will be under pressure if we push a lot of logs to it. Regardless of which solution to choose, we need to ensure that our APIs only log debug or trace on demand to reduce the overall log size. 

The control pane has a tab that can query and update the logging level based on the API exposed from the http-sidecar. If the backend implements the same admin API, we can allow the request to pass through so that the backend logging level can be changed on-demand from the control pane even it is not implemented in the light-4j framework. 

As part of the API platform, we should enforce the backend API to implement the logging level query and update to have fine-grained control about the logging when it is needed and turn it off after the diagnose session. 

