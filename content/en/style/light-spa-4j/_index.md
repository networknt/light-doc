---
title: "Single Page Application"
date: 2018-05-14T22:50:00-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "style"
    weight: 09
weight: 09
aliases: []
toc: false
draft: false
reviewed: true
---

When building APIs that are consumed by Single Page Applications running on browsers, security control is different from service to service calls. This repository contains all the middleware handlers designed for SPA only. It automatically renews the JWT token from the OAuth 2.0 provider before the token expires and handles the CSRF token to prevent cross-site request forgery attacks.

The embedded handler also handles the login and logout from the UI and manages the session with cookies. 

* [Stateless Auth](/style/light-spa-4j/stateless-auth/)

