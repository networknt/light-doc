---
title: "Bootstrap Token"
date: 2021-09-15T11:09:54-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The light-oauth2 and oauth-kafka (enterprise version) token service supports the issuer of long-lived bootstrap token for Platform Components integration. 


The bootstrap token client usually will be the DevOps pipeline, and it will retrieve a bootstrap token during the Kubernetes deployment. 


### Bootstrap Token

Here is an example of token generated from the test case.

```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk0NzA3MzQ5NywianRpIjoiVFY4cndsOWpDMEVVMWtmUWV6MFdHZyIsImlhdCI6MTYzMTcxMzQ5NywibmJmIjoxNjMxNzEzMzc3LCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTAwIiwic2NvcGUiOlsiQThFNzM3NDBDMDA0MUMwM0Q2N0MzQTk1MUFBMUQ3NTMzQzhGOUYyRkI1N0Q3QkExMDcyMTBCOUJDOUUwNkRBMiJdLCJzaWQiOiJjb20ubmV0d29ya250LnBldHN0b3JlLTEuMC4wIn0.TKX_UPjPLkj2AETbDrOyJpPik1qVwU7OzQo4yAqTnesIlqCXW0p7SdJmvnfmgKO-Z6I4YRiroN_MIaRw-pxu5tiKziD_pyrOJJ0KzFQVDKCnVp9nzkbGd64pwH3VQLylTL_wVdU6VdC_pHiBFlvDyAvL9G-Xal1Koe905E_iDJOm5RGhNcm8MQ1VEvIWv31sO7FYKnibjYAnCtTOhHmYHLuHmSe15ID-H_QpC9BHneHLz4CugUZEC2nR5oam_3lmOarNiyNEVG-jv9OmNhJyYffPbSHp7YBCqiC9Z-p3qbTQ-PkxZ5kmZgVrxZlAYkcUZJy2TaJN1PYYARoxlCsdbw

```

Here is the payload of the token. 

```

{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1947073497,
  "jti": "TV8rwl9jC0EU1kfQez0WGg",
  "iat": 1631713497,
  "nbf": 1631713377,
  "version": "1.0",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e00",
  "scope": [
    "A8E73740C0041C03D67C3A951AA1D7533C8F9F2FB57D7BA107210B9BC9E06DA2"
  ],
  "sid": "com.networknt.petstore-1.0.0"
}
```

The token looks similar to other normal tokens, but there are several differences.

* The token is long-lived, and the exp is ten years.
* The client_id is always the same, and it is from the oauth-token.yml config file.
* The scope is always the same, and it is from the oauth-token.yml config file. 
* The sid is the serviceId that is currently deploying, and it is passed from the caller. This custom claim is used for the other party to identify the service from the request. 


### Endpoint

###POST /oauth2/token
Get bootstrap token with client id and secret

####Authorization:

* Using header 
``` 
  Authorization: Basic ZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTAwOmY2aDFGVEk4UTMtN1VTY1BaRHpmWEE=
```

* Using body form
```
client_id="f7d42348-c647-4efb-a52d-4c5787421e00"
client_secret="f6h1FTI8Q3-7UScPZDzfXA"
```

####Required header parameters: 

* Host = localhost
* Content-Type = application/x-www-form-urlencoded
* Transfer-Encoding = chunked

####Required body form parameters:

* grant_type = bootstrap_token
* sid