---
title: "Datasource"
date: 2018-06-17T20:42:27-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Most services have to persist data into a database. Although No-SQL databases are getting popular these days, relational databases are still used by most enterprise customers. We used to create a datasource instance from service.yml automatically just like the below configuration. 

```
- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: org.h2.jdbcx.JdbcDataSource
      jdbcUrl: jdbc:h2:~/test
      username: sa
      password: sa
```

The above config section in service.yml creates an H2 datasource automatically without any code. However, there are several drawbacks. 

* There is only one datasource you can create for your service.
* There is no way you can customize the datasource with its JDBC specific parameters.
* The password is clear text in the service.yml which is not allowed by some organizations.

To resolve above limitations, a new datasource module is created in light-4j:

```
<!-- https://mvnrepository.com/artifact/com.networknt/data-source -->
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>data-source</artifactId>
    <version>${version.light-4j}</version>
</dependency>
```

All common RDBMS datasources are defined in datasource.yml file as following.

datasource.yml

```
# This file gives you several examples for setting up HikariCP with datasource
# module in light-4j. There are two parts in the configuration, properties for
# the HikariCP and properties for the database driver itself. For the parameters
# defined for JDBC driver, put them under parameter with single quote as it is
# handled as string.
MysqlDataSource:
  DriverClassName: com.mysql.jdbc.Driver
  jdbcUrl: jdbc:mysql://localhost:3306/eventuate?useSSL=false
  username: mysqluser
  maximumPoolSize: 2
  connectionTimeout: 500
  parameters:
    useServerPrepStmts: 'true'
    cachePrepStmts: 'true'
    cacheCallableStmts: 'true'
    prepStmtCacheSize: '4096'
    prepStmtCacheSqlLimit: '2048'
    verifyServerCertificate: 'false'
    useSSL: 'true'
    requireSSL: 'true'
MariaDataSource:
  DriverClassName: org.mariadb.jdbc.MariaDbDataSource
  jdbcUrl: jdbc:mariadb://mariadb:3306/oauth2?useSSL=false
  username: mysqluser
  maximumPoolSize: 2
  connectionTimeout: 500
  parameters:
    useServerPrepStmts: 'true'
    cachePrepStmts: 'true'
    cacheCallableStmts: 'true'
    prepStmtCacheSize: '4096'
    prepStmtCacheSqlLimit: '2048'

PostgresDataSource:
  DriverClassName: org.postgresql.ds.PGSimpleDataSource
  jdbcUrl: jdbc:postgresql://postgresdb:5432/oauth2
  username: postgres
  maximumPoolSize: 2
  connectionTimeout: 500

SqlServerDataSource:
  DriverClassName: com.microsoft.sqlserver.jdbc.SQLServerDataSource
  jdbcUrl: jdbc:sqlserver://sqlserver:1433;databaseName=oauth2
  username: sa
  maximumPoolSize: 2
  connectionTimeout: 500

OracleDataSource:
  DriverClassName: oracle.jdbc.pool.OracleDataSource
  jdbcUrl: jdbc:oracle:thin:@localhost:1521:XE
  username: SYSTEM
  maximumPoolSize: 2
  connectionTimeout: 500
  parameters:
    useServerPrepStmts: 'true'
    cachePrepStmts: 'true'
    cacheCallableStmts: 'true'
    prepStmtCacheSize: '10'
    prepStmtCacheSqlLimit: '2048'

H2DataSource:
  DriverClassName: org.h2.jdbcx.JdbcDataSource
  jdbcUrl: jdbc:h2:mem:XE;MODE=MYSQL;INIT=runscript from 'classpath:scripts/test-oauth2.sql'\;runscript from 'classpath:scripts/test-tokenization.sql'\;runscript from 'classpath:scripts/test-vault.sql'\;runscript from 'classpath:scripts/test-todo.sql';
  username: sa
  maximumPoolSize: 2
  connectionTimeout: 500

```

Please note that database specific properties are defined in parameters section which is quoted as String. 

As you can see that we only have usernames in this file, passwords are located in secret.yml for easy management. 

secret.yml

```
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Fresh token client secret for OAuth2 server
refreshTokenClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA


# Consul service registry and discovery

# Consul Token for service registry and discovery
consulToken: token1

# EmailSender password default address is noreply@lightapi.net
emailPassword: change-to-real-password

# dbPassword: CRYPT:MHLp4zqPgyPTQPMU1mBWihFFzoDXimsCXF9GlO55BMo=
dbPassword: mysqlpw

# htPassword: password for the embedded database
h2Password: sa

# Mysql Password
mysqlPassword: mysqlpw

# Maria Password
mariaPassword: mysqlpw

# Postgres Password
postgresPassword: my-secret-pw

# SQL Server Password
sqlServerPassword: StrongPassw0rd

# Oracle Password
oraclePassword: oracle
```

From release 1.6.0, light-4j handler encrypts/decryps in every config file, and so the user can define the encrypted password in the datasource.yml directly with the datasource definition:

```
SqlServerDataSource:
  DriverClassName: com.microsoft.sqlserver.jdbc.SQLServerDataSource
  jdbcUrl: jdbc:sqlserver://sqlserver:1433;databaseName=oauth2
  username: admin
  password: CRYPT:MHLp4zqPgyPTQPMU1mBWihFFzoDXimsCXF9GlO55BMo=
  maximumPoolSize: 2
  connectionTimeout: 500

```

The system will check if there is a password field defined in the datasource or not (use field name ‘password’). If there is no ‘password’ defined, the system will check the secret.yml.

The mapping can be found in the service.yml file so that each datasource instance can be loaded from SingletonServiceFactory. 

service.yml

```
# Singleton service factory configuration

singletons:
- com.networknt.utility.Decryptor:
  - com.networknt.decrypt.AESDecryptor

- javax.sql.DataSource:
  - com.networknt.db.MysqlDataSource::getDataSource:
    - java.lang.String: MysqlDataSource


- com.networknt.db.MysqlDataSource:
  - com.networknt.db.MysqlDataSource:
    - java.lang.String: MysqlDataSource

- com.networknt.db.MariaDataSource:
  - com.networknt.db.MariaDataSource:
    - java.lang.String: MariaDataSource

- com.networknt.db.PostgresDataSource:
  - com.networknt.db.PostgresDataSource:
    - java.lang.String: PostgresDataSource

- com.networknt.db.SqlServerDataSource:
  - com.networknt.db.SqlServerDataSource:
    - java.lang.String: SqlServerDataSource

- com.networknt.db.OracleDataSource:
  - com.networknt.db.OracleDataSource:
    - java.lang.String: OracleDataSource

- com.networknt.db.H2DataSource:
  - com.networknt.db.H2DataSource:
    - java.lang.String: H2DataSource

```

To define multiple datasources with the same database type, the user can define multiple datasources with different dsname:

datasource.yml

```
OauthSqlServerDataSource:
  DriverClassName: com.microsoft.sqlserver.jdbc.SQLServerDataSource
  jdbcUrl: jdbc:sqlserver://sqlserver:1433;databaseName=oauth2
  username: oauth
  password: CRYPT:MHLp4zqPgyPTQPMU1mBWihFFzoDXimsCXF9GlO55BMo=
  maximumPoolSize: 2
  connectionTimeout: 500

EventSqlServerDataSource:
  DriverClassName: com.microsoft.sqlserver.jdbc.SQLServerDataSource
  jdbcUrl: jdbc:sqlserver://sqlserver:1433;databaseName=oauth2
  username: event
  password: CRYPT:MHLp4zqPgyPTQPMU1mBWihFFzoDXimsCXF9GlO55BMo=
  maximumPoolSize: 2
  connectionTimeout: 500

```

Define the mapping in the service.yml:

```
- com.networknt.db.SqlServerDataSource:
  - com.networknt.db.SqlServerDataSource:
    - java.lang.String: OauthSqlServerDataSource
    - java.lang.String: EventSqlServerDataSource

```

And then in the application code, the user can get the datasource list with defined in the service.yml:

```

SqlServerDataSource[] datasources = SingletonServiceFactory.getBeans(SqlServerDataSource.class);


```


To learn how to use it, please take a look at the test case in the datasource module. 

