---
title: "User Integration"
date: 2018-03-03T20:14:50-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

# Integration Test

Get the project from GitHub and build the project:


```
cd ~/networknt
git clone git@github.com/networknt/light-portal.git
cd light-portal/user-management
mvn clean install
``` 

## Normal microservice:

It can be run local to and persist the user info to local database only.

Module:

usermanagement-service

Run docker-compose file for the service:

```
cd ~/networknt/light-portal/user-management

docker-compose up
``` 

The docker-compose will start local postgres database and usermanagement-service microservice



## Event sourcing microservice:

Integrate the service with light-eventuate-4j framework to process user management with event sourcing. It include command side service and query side service.

Module:

rest-coomand    -- command side service

rest-query      -- query side service

Before starting any service, we need to make sure that light-eventuate-4j is
up and running. Please follow this [tutorial][] to set up.

Run docker-compose file for the service:

```
cd ~/networknt/light-portal/user-management

docker-compose -f docker-compose-eventuate.yml up

```

The docker docker-compose-eventuate.yml will start local postgres database (used by query side service) and restful command side service and restful query side service



## Verify result:


-- New user signup

```
curl -X POST \
  http://localhost:8080/v1/user \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
   -d '{"screenName":"testUser","contactData":{"email":"aaa.bbb@gmail.com","firstName":"test1","lastName":"bbb1","addresses":[{"country":"CA","state":"AK","city":"BaBa","addressLine1":"222 Bay Street","addressType":"SHIPPING"}],"gender":"MALE"},"timezone":"CANADA_EASTERN","locale":"English (Canada)","password":"12345678","host":"google","emailChange":false,"passwordReset":false,"screenNameChange":false}
'
```

-- Active user by the link in the confirm email (replace the token from DB)

```
curl -X PUT \
  http://localhost:8080/v1/user/token/0000015e2a49af26-0242ac1200020000 \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json'
```

-- Get ALL user list (for admin user)

```
curl -X GET \
  http://localhost:8080/v1/user \
  -H 'cache-control: no-cache' \
```

--Get user by login in name

```
curl -X GET \
  'http://localhost:8080/v1/user/name?name=testUser' \
  -H 'cache-control: no-cache' \
 ```

--User login

```
curl -X PUT \
  http://localhost:8080//v1/user/login \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{"nameOrEmail":"testUser","password":"12345678"}'
```

-- User change (change email)

```
curl -X PUT \
  http://localhost:8080/v1/user/1080408346077690 \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{"screenName":"testUser","contactData":{"email":"aaa.bbb2@gmail.com","gender":"UNKNOWN"},"timezone":"CANADA_EASTERN","locale":"English (Canada)","emailChange":true,"passwordReset":false,"screenNameChange":false}'
```

-- Possword reset

```
curl -X PUT \
  http://localhost:8081/v1/user/0000015e54ab8932-0242ac1200080000 \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'postman-token: f88f295d-8bcc-dfad-acd3-0d01d65f40b4' \
  -d '{"screenName":"testUser","password":"22222222", "contactData":{"email":"aaa.bbb2@gmail.com","gender":"UNKNOWN"},"timezone":"CANADA_EASTERN","locale":"English (Canada)","emailChange":false,"passwordReset":true,"screenNameChange":false}'
```


-- Login with token (after password reset, system will require use sign again with new password and confirm token)
```
curl -X PUT \
  http://localhost:8082/v1/user/login \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'postman-token: 4493c47b-3f83-5c0b-81ed-7032178ca5f6' \
  -d '{"nameOrEmail":"testUser","password":"22222222", "token": "0000015e54c0d4b1-0242ac1200070000"}'
```



## End

[tutorial]: /tutorial/eventuate/getting-started/
