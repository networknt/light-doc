---
title: "Marketplace Auth"
date: 2018-07-27T11:46:50-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Marketplace Auth provider is an example that shows how to create your customized auth provider. It covers very complicated scenarios as follows.

- It authenticates the internal user with Kerberos/SPNEGO single sign-on and falls back to Basic Authentication with LDAP. 

- For the external user, it goes to the user database to verify the userId and password for authentication.

- For both internal and external users, the roles will be retrieved from a file in github.com site. If the user cannot be found in the file, a default role will be assigned. 

