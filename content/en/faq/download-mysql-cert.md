---
title: "Download MySQL Cert from Docker Container"
date: 2018-06-17T11:02:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When using an event-based framework, we need to use MySQL Binlog library to connect to MySQL server to access the trailing transaction log. To make the connection secure, TLS will be used by the client. 

If you have a DBA install the MySQL instance for your application, you can ask for the client certificate from him/her. However, if you are using MySQL docker for your testing, you need to get the client certificate from the docker container. Here are the steps to get the client cert installed. 


### Copy client.truststore

The first step is to copy the client.truststore and client.keystore into your config/tls folder from client module in light-4j. In our case, we might need to copy the tls folder to light-tram-4j/tram-cdc-mysql-server/src/main/resources/config folder.

### Download client certificate

If you are using the official MySQL docker image, the certificate files are located inside the contain at
/var/lib/mysql folder. Go to the above folder you have just copied client.truststore file.

```
docker cp <containerId>:/var/lib/mysql/ca.pem .
docker cp <containerId>:/var/lib/mysql/client-cert.pem .
```

Now we need to import both ca.pem and client.pem into the client.truststore.

```
keytool -v -import -file client-cert.pem -alias mysqlcrt -keystore client.truststore 

```
