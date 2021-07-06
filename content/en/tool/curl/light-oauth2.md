---
title: "Light Oauth2 Curl Commands"
date: 2020-04-23T17:05:37-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---


Post to /oauth2/code

```
curl -k --location --request POST 'https://devsignin.lightapi.net/oauth2/code' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: __cfduid=d1beeb242d986229165bc27809633088a1585943573' \
--data-urlencode 'client_id=f7d42348-c647-4efb-a52d-4c5787421e72' \
--data-urlencode 'user_type=customer' \
--data-urlencode 'state=1222' \
--data-urlencode 'remember=N' \
--data-urlencode 'j_username=stevehu@gmail.com' \
--data-urlencode 'j_password=123456'
```
