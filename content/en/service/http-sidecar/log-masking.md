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
* Need to manage the mapping for each service to its namespace on Splunk.

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


### Transfer logs to sidecar

If we want to http-sidecar to handle the log masking on behalf of the backend API, we need to find a way to transfer the backend logs to the http-sidecar. There are two options, and each option has its merits 

One way is to write the backend log to a volume that is shared to the http-sidecar, and the sidecar picks the file periodically to merge the log with the sidecar log, which Splunk will pick up. The masking will be done during this merging process based on the mask.yml configuration file on the sidecar. 


Pros: 

* There is no backend logging configuration change, but we need to change the overlay to specify another location for the backend logs. 
* Suitable for all backend API frameworks and languages as all logging frameworks will support file system appenders. 
* Better performance and resource utilization for the backend API as writing into a file is faster than sending an HTTP request. 
* Can handle huge log volumes without significant performance degradation.
* There is no need to update the http-sidecar to expose an extra API for log injection.

Cons:

* The sidecar needs to have a startup hook to periodically scan the backend logs and merge the logging statements into the sidecar logs.
* If any un-caught exception happens, the thread might be dead, and it is hard to get it restarted without restarting the server. That means we have to test the implementation thoroughly to ensure all possible runtime exceptions are handled gracefully. 
* Given the nature of the batch process, it is tough to track which file and which line of backend API logs are processed, and the merging logic is a little bit complex due to the timestamp comparison. 


The other way is to pick a logging framework that supports the SocketAppender to send the logs to the sidecar in batch asynchronously. 

Pros:

* The SocketAppender provided by the logging framework should be tested and stable. 
* A runtime exception won't stop the entire process, so runtime exceptions are recoverable. 

Cons:

* Not suitable to handle huge volumes as the same HTTP channel is shared with other HTTP requests. 
* Not all logging frameworks support the SocketAppender, so some project teams might be forced to switch to another logging framework. 
* We need to enhance the sidecar to expose an API to accept back-end API logs. 


We might need to analyze the current portfolios before deciding on which solution. Also, it might be a good idea to start the PoC to verify the two solutions before settling down on the right one.


### Masking Config Management

Regardless of how we pass the logs to the http-sidecar from the backend API, the http-sidecar does the masking based on the mask.yml configuration file designed per API based on the data it is handling. 

In a big project team, most developers don't need to worry about the log masking, and the team lead will update the mask.yml during the code review and log review during the development cycle. 

In the testing cycle, the testers will test the masking by reviewing the generated logs with the maximum logging level enabled. Any sensitive data that shows up in the log will be marked as a defect. 

If Kafka is used for piping the logs to Splunk, we will put a microservice to stream process the logs to seek some patterns like SIN, credit card number, email etc. The warning captured might be a false alarm, and it needs the project team to analyze the result to decide to add a new mask rule or not. The result can be utilized for the certification for the API to indicate if a particular API has masked all possible logging statements or not. 

For enterprise users who are leveraging the light-portal schema registry, we can extract all common masking rules from it. It requires all APIs to register their JSON Schemas in their specification to the schema registry and only refer to it remotely. Once a new API is designed, they can go to the schema registry to look up the standard object definition instead create a new one. Mask.yml will have a common section defined for all APIs and an API-specific section with this approach. 

### Merging the http-sidecar and Kafka sidecar

If we decide to use Kafka for logging ingestion, the http-sidecar will access the enterprise Kafka cluster. In this case, it might be easier for us to merge the http-sidecar and kafka-sidecar to provide only one docker image to the backend API. All features in the sidecar can be enabled or disabled by changing the configuration from the config server. Here are the pros and cons of having a separate Kafka sidecar. 

Pros:

Cons:


Here are the pros and cons of having only one sidecar for both HTTP and Kafka cross-cutting concerns. 


Pros:


Cons:

* 








