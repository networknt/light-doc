---
title: "Config Server API"
date: 2018-07-04T14:48:25-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

There are four groups APIs for the config server. 

### Config Server Admin - Initialize Server

This API is used to initialize the server before using it. Two keys must be populated from the UI by two different operators on two different pages and submitted to the API. Once the server receives these keys, it will combine them together and generate a hash that is used to be the encryption key for the sensitive values in the database. 

If there are multiple instances of config server running, this initialize process must be done on each server container. In most cases, one instance should be enough and you can setup one active and one backup for disaster recovery. If you want multiple service that all active at the same time behind a proxy, then you need to ensure that all instances are initialized the same way. 

There is another option that we can skip the UI but calling the API directly with curl from the command line. In this case, the two users need to get an authorization code grant type JWT token to put into the request header and each user will call the API once with his/her key part. The current API accepts two key parts all together and it might need to be changed.

### Service Owner - Serivce Maintenance

This API has several endpoint to create/update/delete/query service(s) and it is used by mainly by the service owner role. 

The service owner can create a new service on the config server and specify algo, salt and template repository. 





