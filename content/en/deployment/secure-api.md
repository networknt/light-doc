---
title: "Secure API"
date: 2019-01-15T07:18:40-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "deployment"
    weight: 41
weight: 41
aliases: []
toc: false
draft: false
reviewed: true
---

After you complete your service development, you need to consider how to deploy your service to the production with security in mind. In this section, we will discuss some of the security measurements that help you to prevent attacks from the Internet when you expose your service to the public.

Services built on top of light-4j are inherently more secure than most platforms as we have a lot of middleware handlers to address cross-cutting concerns and a significant number of them are for security. That means attack requests must pass all security middleware handlers to reach your business logic. That being said, there are still a lot of extra mechanisms that can be implemented to maximize your service security.

Please note that this article is for small to medium organizations that don't have access to CDN and F5. For enterprise-level security, please contact our support team at support@networknt.com

### The light-router

In almost all scenarios, we are not going to expose our service to the Internet directly, especially if your service is deployed to the cloud with Kubernetes. In fact, you cannot expose these services directly as they don’t have a static IP and port number.

Most of our users are using light-router for their external access point which is deployed on a VM with a static IP on port 443 only. The light-router is responsible for security, service discovery, serving single page applications, etc. For more information, please visit [light-router document][] and [light-router tutorial][].

### Cloudflare

To prevent a DDoS attack, it is wise to hide the real IP address by leveraging some CND services. Cloudflare is one of the best, and it provides DNS service for free. Just move your domain DNS to Cloudflare so you can hide your IP address from the Internet. You can find a lot of information about security in the [security tutorial][].

### Security Middleware

There are a lot of security-related middleware handlers shipped in Light, and you should consider enabling some of them on your light-router instance or your services. The following are the most commonly used.

* [JWT Verifier](/style/light-rest-4j/openapi-security/)
* [IP Whitelist](/concern/ip-whitelist/)
* [Rate Limit](/concern/limit/)
* [CORS](/concern/cors/)


[light-router document]: /service/router/
[light-router tutorial]: /tutorial/router/
[security tutorial]:/tutorial/security/


