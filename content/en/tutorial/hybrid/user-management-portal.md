---
title: "User Management Portal"
date: 2018-03-21T06:46:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

User management is the hybrid service in light-portal which used for portal user management. It provides the following services:

-- Register New User

-- Active User (By click the link in the email to entry the link from browser)

-- Change User contact information

-- Change User email address (will send email to confirm email change)

-- Change User login Id

-- Delete User

-- Login User (integrate with light-oauth2 service to get user token and integrate with light-session for session management)

-- Get User detail by Id

-- Get User detail by email

-- Get User detail by login Id

-- Get All Active User (admin role)



User management Hybrid service is entry point for light-portal. After user Register User and login to the portal, user can start to use the ALL portal services which include Host-Menu, Schema-Form, Light-Oauth...




### Build services

Build user-management project in the light-portal.

```
cd ~/networknt/light-portal/user-management
mvn clean install
```


### Setting and running hybrid-service

Build light-portal common-util and hybrid-command, hybrid-query service

```
cd ~/networknt/light-portal/
mvn clean install
```


Now let's copy the user-management service jar and lib jar files to the service folder for the hybrid-command in
light-config-test.

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/service
cp ~/networknt/light-portal/user-management/auth/target/usermanagement-auth-0.1.0.jar .
cp ~/networknt/light-portal/user-management/common/target/user-management-common-0.1.0.jar .
cp ~/networknt/light-portal/user-management/jdbc/target/usermanagement-jdbc-0.1.0.jar .
cp ~/networknt/light-portal/user-management/hybrid-service/target/user-hybrid-service-0.1.0.jar .
``` 

Use the following command to start the server.

```
cd ~/networknt/light-config-test/light-portal/hybrid-command/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-command/target/hybrid-command-1.5.11.jar:../service/* com.networknt.server.Server
```


### Verify (with sample curl url)

-- Create New User

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d ' {"host":"lightapi.net","service":"user","action":"createUser","version":"0.1.0","data":{"host":"networknt.com","screenName":"testUser","password":"1222222","contactData":{"email":"gavinyan.han@gmail.com","addresses":[{"country":"CA","state":"AK","city":"BaBa","addressLine1":"222 Bay Street","addressType":"SHIPPING"}],"gender":"MALE"}}}
'

```

After user is created, the user should get an email from 'lightapi.net', to active the user.

The user can click the link or get the url from the link and manually input into browser to active user.


```

Welcome to Light-portal!

To complete your account registration, please confirm your email address by clicking on the following link:

Click <link> to activate your account.

If you cannot click on the link above, please copy and paste it into your web browser.
Please visit https://github.com/networknt/light-portal for more information.

```


-- Update User

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d ' {"host":"lightapi.net","service":"user","action":"updateUser","version":"0.1.0","data":{"id":"222-111", "host":"networknt.com","screenName":"testUser","password":"1222222","contactData":{"email":"lqqq.qqq@DDD.COM","addresses":[{"country":"CA","state":"AK","city":"BaBa","addressLine1":"222 Bay Street","addressType":"SHIPPING"}],"gender":"MALE"}}}
'

```

— DeleteUserById

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"user","action":"deleteUserById","version":"0.1.0","data":{"id":"111222-2222"}}'

```

-- Login User

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"user","action":"loginUser","version":"0.1.0","data":{"nameOrEmail":"testUser", "password": "1222222"}}'
```


-- Get Users (Admin role only)

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"user","action":"getUsers","version":"0.1.0"}'
```



— GetUserByEmail:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"user","action":"getUserByEmail","version":"0.1.0","data":{"email":"gavinyan.han@gmail.com"}}'
```


— GetUserByName

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"user","action":"getUserByName","version":"0.1.0","data":{"name":"testUser"}}'
```
