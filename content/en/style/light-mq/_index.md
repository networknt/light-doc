---
title: "Light IBM MQ"
date: 2020-01-17T22:42:27-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 11
weight: 11
aliases: []
toc: false
draft: false
reviewed: true
---

This is not an open-source component, so it is only available for paid customers. We have some enterprise customers who still have so many applications or services built on top of the Java EE platform with IBM MQ to integrate between applications across domains. To leverage these applications and services, we were asked to develop a light-4j component that can easily connect to the MQ to produce/consume messages. 

### Features

* Security first design with all connections authenticated and authorized.
* TLS support
* Automatically create the JMSContext during the server startup and close the context during the server shutdown. 
* MQConfig and JMSContext are exposed through the startup hook for app developers.
* MQ component can be enabled or disabled by an external config file

### Configuration

Here is an example of mq.yml configuration file

```
# If the MQ component is enabled or not
enabled: true
# If TLS is used to connect to the MQ server
loadTrustStore: true
# The Truststore that contains the MQ client certificate
trustStore: mq.truststore
# The password that is used to open the truststore
trustStorePass: passw0rd
# Queue Manager
queueManager: QM1
# Channel
channel: DEV.APP.SVRCONN
# Connection host
host: localhost
# Connection port
port: 1414
# user
user: app
# password
password: passw0rd
# Cipher Suite used for the TLS
cipherSuite: SSL_RSA_WITH_AES_128_CBC_SHA256
# Default Queue Name
queue: DEV.QUEUE.1
# Default Topic Name
topic: dev/
```

### Tutorial

There are two examples in the component to show users how to use the light-mq. One example is a producer that has two endpoints to put request body into a queue and to publish request body to a topic. Another example is a consumer service that has two startup hooks to consume the message from the queue and the topic in separate threads. It has two endpoints to display the last ten messages received from the queue and the topic. 

The source code of the example applications is not available as open-source; however, we have published the docker images to the docker hub so that users can run the application locally by following the [tutorial](/tutorial/mq/). 

### License

If anyone is interested in using this component, please contact sales@lightapi.net for more information. 

### Support

If you are a customer and have questions or issues with the component, please contact support@lightapi.net for support. For customers who have other support channels, please use them for faster response than email support. 

