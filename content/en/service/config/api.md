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

If there are multiple instances of config server running, this initialize process must be done on each server container. In most cases, one instance should be enough and you can setup one active and one backup for disaster recovery. If you want multiple services that all active at the same time behind a proxy, then you need to ensure that all instances are initialized with the same way. 

There is another option that we can skip the UI but calling the API directly with curl from the command line. In this case, the two users need to get an authorization code grant type JWT token to put into the request header and each user will call the API once with his/her key part. The current API accepts two key parts all together and it might need to be changed.

### Service Owner - Service Maintenance

This API has several endpoints to create/update/delete/query service(s) and it is used  mainly by the service owner role. 

The service owner can create a new service on the config server and specify algo, salt and template repository. 


### Value Operator - Key/Value Maintenance

This API has several endpoints to create/update/delete/query common key/value pairs that are shared across multiple services. Also, it has several endpoints to create/update/delete/query service specific key/value pairs. As these values are not sensitive, the security level for these operators is not high. 

The config value will be save into data store (NoSql DB or RDBMS DB). The config value will have two type entries:

-- Service specific config value(identify by service Id)

-- Common config value, which is same for all services in the environment.


From config value key, system identify the template and and config name in the config template file.

For example:

config key:  service/javax.sql.DataSource/com.zaxxer.hikari.HikariDataSource/DriverClassName

It will identify following config in the service.yml file (template):

javax.sql.DataSource
  com.zaxxer.hikari.HikariDataSource
    DriverClassName



### Secret Operator - Key/Secret Maintenance

This API has several endpoints to create/update/delete/query common key/secret pairs that are shared across multiple services. Also, it has several endpoints to create/update/delete/query service specific key/secret pairs. As these secret are sensitive, the security level for these operators is high. 

When the Config Server service started, system will create an encrypt/decrypt key file on the files system (by default will be in the user home folder).

 File name:  light-config-server.conf


System admin should trigger the Initial server service first after light-config-server service start to run to set the encrypt/decrypt key into the key file ( light-config-server.conf)

Above basically defines four different roles in the config server. When light-oauth2 is integrated with the AD, these roles can be retrieved from the LDAP. Or the roles can be managed by the light-oauth2 user table if that is desired.



### Config Server API workflow diagram

![Provider workflow](/images/light-config-server.png)
