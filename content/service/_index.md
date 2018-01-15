---
title: "Service Overview"
date: 2017-11-04T22:12:19-04:00
description: "There are some services as part of the infrastructure to support microservices"
categories: [service]
keywords: []
menu:
  docs:
    parent: "service"
    weight: 01
weight: 01	#rem
aliases: []
toc: false
draft: false
---

For organizations that is planning to migrate existing monolithic application to
microservices, the up front infrastructure investment is very significant. There
are some key services must be implemented before deploying large number of services.

Normally, you can start building several services and deploy them to your existing
data center; however, once you have dozen services running with multiple instances
you will need supporting services for security, monitoring, metrics etc.

The following are some of the important services recommended by light platform. Some
of them are built by ourselves as existing products on the market cannot meet our
requirement with microservices in cloud. For example, light-oauth2 and light-portal.
Other services are our picks of industry leaders and the list is changing as the
landscape of cloud native computing is changing dramatically. 

* [light-oauth2][] is our security service built on top of light-rest-4j to protect
services accessed by unauthorized client.

* [light-portal][] is an API runtime management and marketplace.

* [light-proxy][] is a proxy service that can bring legacy REST API to light platform
ecosystem by providing all cross-cutting concerns as embedded gateway.

* [Messaging service][] is provided by [Apache Kafka][] for high throughput and high
availability. 

* [Logging][] is provided by [ELK][] stack for centralized logging

* [Metrics][] is provided by [InfluxDB][] or [Prometheus][] with [Grafana][] as front-end.

[light-oauth2]: /service/oauth/
[light-proxy]: /service/proxy/
[light-portal]: /service/portal/
[Messaging service]: /service/messaging/
[Apache Kafka]: https://kafka.apache.org/
[Logging]: /service/logging/
[ELK]: https://www.elastic.co/webinars/introduction-elk-stack
[Metrics]: /service/metrics/
[InfluxDB]: https://www.influxdata.com/
[Prometheus]: https://prometheus.io/
[Grafana]: https://grafana.com/




  
  

