---
title: "Tokenization Roadmap"
date: 2018-03-25T11:02:42-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

### Database Encryption

For some organizations, they need to encrypt the value in the token_vault table so that even the database is compromised, the values linked to tokens are still useless for hackers. 

As tokenization service is read heavy and we have an in-memory cache that contains clear text. The read service performance won't be impacted by the database encryption.  The token creation may be a little bit slower than clear text as the encryption is part of the process. 

The encryption is configurable at the service level first. And then at each format scheme, there is a flag to support encryption or not. In this case, not all data is encrypted. There must be a prefix for encrypted value in database value to indicate that. 

### Token Value Mapping

When creating a new token, there are two different approaches: 

* Search into the vault to find if the same value got a token already and return that token. 

* Blindly create a brand new token a for the value regardless if the value exists in the vault or not. 




### Token Expiration



### Token Revocation



