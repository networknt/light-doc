---
title: "Light Proxy Tutorial"
date: 2017-12-16T09:36:38-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Light-proxy is a critical service component that can glue all other alien APIs into the Light ecosystem without re-writing. Once putting light-proxy in front of legacy APIs, these API can enjoy all the cross-cutting concerns provided by the light-platform, such as security, metrics, auditing, traceability, validation, service registry/discovery, etc. From the consumer point of view, your backend API looks just like an API built on top of light-4j frameworks. 

### OpenAPI 3.0 Backend 

* [OpenAPI backend service][] to simulate legacy RESTful service built with OpenAPI 3.0 spec. 

* [OpenAPI proxy][] to set up light-proxy in front of OpenAPI 3.0 service with OpenAPI 3.0 spec.

### Docker

* [Proxy Docker][] will show you how to use dockerized proxy with externalized config files

### Confluent Schema Registry

* [Schema Registry][] shows how to give read-only access to Internet applications and give write access to internal applications 

### Enable Metrics on Proxy

* [Proxy Metrics][] shows how to enable metrics on the proxy server with InfluxDB as the database. 

[OpenAPI backend service]: /tutorial/proxy/openapi-backend/
[OpenAPI proxy]: /tutorial/proxy/openapi-proxy/
[Proxy Docker]: /tutorial/proxy/docker/
[Schema Registry]: /tutorial/proxy/schema-registry/
[Proxy Metrics]: /tutorial/proxy/proxy-metrics/


