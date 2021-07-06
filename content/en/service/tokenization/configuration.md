---
title: "Tokenization Service Configuration"
date: 2018-03-22T22:01:36-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In most of the cases, the only configuration file you want to change is datasouce.yml file. 

The default datasource.yml file packaged into the docker container: 

```yml
tokenization:
  jdbcUrl: jdbc:mysql://mysqldb:3306/tokenization?useSSL=false
  username: mysqluser
  password: mysqlpw
  parameters:
    maximumPoolSize: 10
    useServerPrepStmts: true
    cachePrepStmts: true
    cacheCallableStmts: true
    prepStmtCacheSize: 40
    prepStmtCacheSqlLimit: 20
vault000:
  jdbcUrl: jdbc:mysql://mysqldb:3306/vault000?useSSL=false
  username: mysqluser
  password: mysqlpw
  parameters:
    maximumPoolSize: 10
    useServerPrepStmts: true
    cachePrepStmts: true
    cacheCallableStmts: true
    prepStmtCacheSize: 40
    prepStmtCacheSqlLimit: 20

```

You might want to change the database jdbcUrl for both connections. Moreover, you might want to add a new vault for a brand new LOB. 

