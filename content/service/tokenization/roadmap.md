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

* Blindly create a brand new token for the value regardless if the value exists in the vault or not. 

It would work if some scheme formats are defined as one to one mapping and other scheme formats are defined as one to many mapping. 


### Token Clean Up

If one value can generate a new token every time the tokenizer is called, then the token space will soon run out. A cleanup strategy must be in place to ensure that unused tokens are removed. We cannot rely on the clients to actively remove a token when they know a token is not used. 

The following two strategies are adopted: 

* One time token. Once the token is read, it is removed. 
* Long-lived token. supposed to be saved in client's database for a long term. 

For the first case, we can generate token randomly every time but for the second use case, we have to generate the same token for the same value in order to reserve the token space. 

### Integration or Batch

For migrating an existing application, a batch endpoint needs to be provided so that client can call the endpoint to pass in 1000 values to generate 1000 tokens in a batch to update its database in one transaction. The number of the tokens can be flexible but a maximum number should be limited in order for the service instance to still serve other clients. 


### Token Revocation

In case a token is compromised, an endpoint needs to be provided to revoke the token so that nobody can call the service to get the true value. We cannot remove the record from the database though as that particular token might be persisted in another database. A special scope and endpoint can be used to retrieve the token if necessary. 




