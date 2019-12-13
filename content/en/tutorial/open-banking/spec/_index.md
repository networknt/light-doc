---
title: "API Specification"
date: 2019-12-11T14:18:55-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The specifications are based on the [Account and Transaction API Specification][] from OpenBankingUK. And the original repository can be found at https://github.com/OpenBankingUK/account-info-api-spec

The original specification is Swagger 2.0, and it is designed as a monolithic API with one spec. We first convert it to OpenAPI 3.0 format and then break it up into several microservices. 

[Account and Transaction API Specification]: https://openbanking.atlassian.net/wiki/spaces/DZ/pages/1077805296/Account+and+Transaction+API+Specification+-+v3.1.2

Open Banking specification contains too many endpoints, and we cannot implement all microservices but only 4 of them for demonstration purposes. 


### accounts

The specification can be found in the [accounts](https://github.com/open-banking/model-config/tree/master/ob/accounts) folder of model-config repository. 

### balances

The specification can be found in the [balances](https://github.com/open-banking/model-config/tree/master/ob/balances) folder of model-config repository. 

### parties

The specification can be found in the [parties](https://github.com/open-banking/model-config/tree/master/ob/parties) folder of model-config repository. 

### transactions

The specification can be found in the [transactions](https://github.com/open-banking/model-config/tree/master/ob/transactions) folder of model-config repository. 

