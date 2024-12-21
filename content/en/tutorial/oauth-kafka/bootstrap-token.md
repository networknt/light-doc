---
title: "Bootstrap Token Tutorial"
date: 2021-09-15T12:34:24-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Checkout this repo to your `~/networknt` folder 

```
cd ~/networknt
git clone git@github.com:lightapi/oauth-kafka.git
```

To start the token server on the local 

```
cd ~/networknt/oauth-kafka/oauth-token
mvn clean install -Prelease
java -jar target/oauth-token-2.0.31-SNAPSHOT.jar
```

To retrieve bootstrap token, call the endpoint

Client id and secret in body form
```
curl -k --location --request POST 'https://localhost:6882/oauth2/token' \
-H 'Host: localhost' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-H 'Transfer-Encoding: chunked' \
-d 'grant_type=bootstrap_token' \
-d 'sid=com.networknt.petstore-1.0.0' \
-d 'client_id=f7d42348-c647-4efb-a52d-4c5787421e00' \
-d 'client_secret=f6h1FTI8Q3-7UScPZDzfXA'
```

OR  

Client id and secret in header

```
curl -k --location --request POST 'https://localhost:6882/oauth2/token' \
-H 'Host: localhost' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-H 'Transfer-Encoding: chunked' \
-H 'Authorization: Basic ZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTAwOmY2aDFGVEk4UTMtN1VTY1BaRHpmWEE=' \
-d 'grant_type=bootstrap_token' \
-d 'sid=com.networknt.petstore-1.0.0'
```
