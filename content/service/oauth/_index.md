---
title: "Light OAuth2"
date: 2017-11-10T12:57:03-05:00
description: ""
categories: [service]
keywords: [oauth2]
menu:
  docs:
    parent: "service"
    weight: 30
weight: 30
aliases: []
toc: false
draft: false
---

The light platform follows security first design. Given the complexity of microservices architecture, there is no existing OAuth 2.0 provider on the market that is suitable for microservices. Most commercial products are focusing on the perimeter and they don't have solutions for the service to service communication. This is the reason we have provided an OAuth 2.0 provider light-oauth2 which is based on light-4j and light-rest-4j frameworks. The light-oauth2 consists of 7 microservices that can be deployed in one cluster with multiple instances of token service and a single instance of other services. Or you can deploy two clusters if your organization needs one cluster for external clients and another cluster for internal clients. 

The light-oauth2 is not just an OAuth 2.0 provider. Some of the services implement the OAuth 2.0 specifications, and others implement some extensions to make OAuth 2.0 more suitable to protect service to service communication regardless request/response (light-rest-4j, light-graphql-4j and light-hybrid-4j) or event-driven (light-eventuate-4j, light-tram-4j and light-saga-4j). It has entitlement management built-in to support scope verification at endpoint level for services with JWT. It also can be federated to support multiple clusters in the same organization. The key distribution service enables resource servers to pull public key certificates that are used to verify JWT from OAuth 2.0 provider instead of traditional push approach. 


- [Why this OAuth 2.0 Provider](/service/oauth/why-this-oauth/)
- [Introduction](/service/oauth/introduction/)
- [Getting started](/getting-started/light-oauth2/)
- [Architecture](/service/oauth/architecture/)
- [Documentation](/service/oauth/service/)
- [Tutorial](/tutorial/oauth/)
- [Reference](/service/oauth/reference/)

### Why this OAuth 2.0 Provider

This is a unique product that is designed for microservices as microservices. It has more features than what are described in popular OAuth 2.0 related specifications. To learn more about the attributes of this product, please refer to [why this OAuth 2.0 provider][]. 

### Getting started

The easiest way to start using light-oauth2 in your development environment is through docker-compose in light-docker repository. Please refer to [getting started][] for more information. If you don't want to run light-oauth2 locally, you can use our [cloud services][]for development. 

### Architecture

There are some key decision-making points that are documented in the [architecture][] section.

### Documentation

The detailed [service document][] helps users to understand how each individual service works and the specification for each service. It also contains information on which scenarios will trigger what kind of errors. 

### Tutorial

There are [tutorials][] for each service that show how to use the most common use cases with examples. 

### Reference

There is a vast amount of information about OAuth 2.0 specifications and implementations. Here are some of the [references][] that can help you to understand OAuth 2.0 Authorization.


[light-4j]: https://github.com/networknt/light-4j
[light-oauth2]: https://github.com/networknt/light-oauth2
[light-portal]: https://github.com/networknt/light-portal
[light-oauth2 service]: /service/oauth/service/
[light-oauth2 tutorial]: /tutorial/oauth/
[getting started]: /getting-started/light-oauth2/
[architecture]: /service/oauth/architecture/
[service document]: /service/oauth/service/
[tutorials]: /tutorial/oauth/
[references]: /service/oauth/reference/
[introduction]: /service/oauth/introduction/
[key distribution]: /architecture/key-distribution/
[cloud service]: /lightapi.net
[why this OAuth 2.0 provider]: /service/oauth/why-this-oauth/
