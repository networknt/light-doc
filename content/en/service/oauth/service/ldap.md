---
title: "LDAP Identity Manager"
date: 2018-06-01T11:20:04-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In most cases, when basic authentication is used, we are going to look up the user profile from a database to verify the hashed/salted password in order to authenticate the user with credentials. For some organizations, they are using Active Directory for user management and everybody logs into a Windows domain. 

In this environment, it is better to setup SPNEGO/Kerberos so that SSO on the broswer can be done seamlessly. If SPNEGO is not implemented, we can still use LDAP to do the SSO explicitly. This LDAP Identity Manager is designed for this use case. 






### Reference

https://connect2id.com/products/ldapauth/auth-explained



