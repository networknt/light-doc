---
title: "Token"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: [oauth2]
keywords: []
weight: 20
aliases: []
toc: false
draft: false
---

This service has only one endpoint to get access token. It supports authorization
code grant type and client credentials grant type. Also, it supports some customized
authorization types required by our customer. These customized flows will only work
on certain clients which have to be setup as trusted clients. 

Authorization Grant with authorization code redirected in the previous step. Please
replace code with the newly retrieved one as it is only valid for 10 minutes.
 
```
curl -k -H "Authorization: Basic f7d42348-c647-4efb-a52d-4c5787421e72:f6h1FTI8Q3-7UScPZDzfXA" -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "grant_type=authorization_code&code=c0iAfPAeTk2BpiPWj-CYPQ" https://localhost:6882/oauth2/token
```

The above command will have the following output.

```
{"access_token":"eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTQ4MzIwOTQ4NiwianRpIjoib0dtdXEzSl85d0tlOUVIT2RWM21PUSIsImlhdCI6MTQ4MzIwODg4NiwibmJmIjoxNDgzMjA4NzY2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6ImFkbWluIiwidXNlcl90eXBlIjoiYWRtaW4iLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJwZXRzdG9yZS5yIiwicGV0c3RvcmUudyJdfQ.gQ5HI2drObxorsQvz86RYT5tgk7QCnEBm9zNod7SbC--v8s4OfFIM4FQbxGqlMzbU3_dDXiyMSGzOFD_ShZ5se9W2FLxLjbMmBJwQG89peymcdY2mTgQoKJMYxL602a7cloyuoDZ_l-OQSj6RMdgRw4FKmMdOqMKWauoh58faZqvHgGxk43hlKW4bBy4vqg2IhNsUm_vIf-SVAUAMqp0Birt94FfjM3QSCQfwHXfK1nCWjFvfRIoN6w7XrPDQtnZq_8Mhdv8dNwowDLoYayKoUpr7i84gFA11-J1gocJOALj1kYody6kU5CfMwGOSX90PUEmdVy_3WnyEAp3blC-Iw","token_type":"bearer","expires_in":600} 
```

Client Credentials grant doesn't need authorization code but only client_id and
client_secret. Here is the curl command line to get access token.

```
curl -k -H "Authorization: Basic f7d42348-c647-4efb-a52d-4c5787421e72:f6h1FTI8Q3-7UScPZDzfXA" -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "grant_type=client_credentials" https://localhost:6882/oauth2/token
```

The above command will have the following output.

```
{"access_token":"eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTQ4MzIwOTc1NywianRpIjoiOVhWdGV2dXZ2cjMwQ0lnZVFuUTFUUSIsImlhdCI6MTQ4MzIwOTE1NywibmJmIjoxNDgzMjA5MDM3LCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOlsicGV0c3RvcmUuciIsInBldHN0b3JlLnciXX0.C8oHgjKpaKWAYJvSqZ4_VT2sw8XXpABFq-aXgNUN2mCEKZJN7AkA6qio0fK4ZCTn5lT9bLou6SOEDV-uXvcU1_XlvKTTnbMO2g-s_7-O-xXxSCAXiLZ-5C7ieGt7enQrxrESUEsgr0Kow4a34GjxAod5j0vcKzhZ6vrcQcuCecPKaeovV0nkBZH2cGPhaLvK346RA9VjxITcR1DgzPWIO3AYJGaIrF8-mCA6Ad8LNi8mB0T5pHIST5fpVTsDYF3KjQJKYiwEhVMbfErBrsmiUUHJ7fYNi5ntLvT-61rupqrQeudl54gg4onct6rT9A2HmuV0iucECkwm9urJ2QxO-A","token_type":"bearer","expires_in":600}
```

If you are interested, you can compare the claims of above tokens at https://jwt.io/

