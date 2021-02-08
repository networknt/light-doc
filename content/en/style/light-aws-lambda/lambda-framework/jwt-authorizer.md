---
title: "Jwt Authorizer"
date: 2021-02-07T15:34:57-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


This custom authorizer implementation verifies tokens in both Authorization and optional X-Scope-Token headers. As the authorizer doesn't know the specification, the scope verification cannot be done in this Lambda function. 

This Lambda function verifies the token signature and expiration only. Also, it enriches the context with necessary claims so that the scope-verifier module can verify the scopes within the Lambda function.
