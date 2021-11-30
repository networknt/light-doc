---
title: "Fine Grained Authorization"
date: 2021-11-22T21:28:26-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

### Light-portal

To run the demo application, you need to start the light-portal locally by following the README.md in https://github.com/lightapi/portal-config-loc


### Upload Rule

Once the portal is up and running, you need to add the rule from the publish/YAML Rule menu. 

```
account-cc-group-role-auth:
  ruleId: account-cc-group-role-auth
  host: lightapi.net
  description: Role-based authorization rule for account service and allow cc token and transform group to role.
  conditions:
    - conditionId: allow-cc
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.user_id
      operatorCode: NIL
      joinCode: OR
      index: 1
    - conditionId: manager
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.groups
      operatorCode: CS
      joinCode: OR
      index: 2
      conditionValues:
        - conditionValueId: manager
          conditionValue: admin
    - conditionId: teller
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.groups
      operatorCode: CS
      joinCode: OR
      index: 3
      conditionValues:
        - conditionValueId: teller
          conditionValue: frontOffice
    - conditionId: allow-role-jwt
      variableName: auditInfo
      propertyPath: subject_claims.ClaimsMap.roles
      operatorCode: NNIL
      joinCode: OR
      index: 4
  actions:
    - actionId: match-role
      actionClassName: com.networknt.rule.FineGrainedAuthAction
      actionValues:
        - actionValueId: roles
          value: $roles

```

### Start access-control

The demo service is located at https://github.com/networknt/light-example-4j/tree/master/rest/access-control

### Test

##### Client Credentials API to API

This is API to API call, no role-based authorization will be applied. 

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1952900650,
  "jti": "3eUw7KQzcNyzswz99_DVBw",
  "iat": 1637540650,
  "nbf": 1637540530,
  "version": "1.0",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e73",
  "scope": "account.r account.w"
}
```


```
curl -k --location --request GET 'https://localhost:8080/accounts' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk1MjkwMDY1MCwianRpIjoiM2VVdzdLUXpjTnl6c3d6OTlfRFZCdyIsImlhdCI6MTYzNzU0MDY1MCwibmJmIjoxNjM3NTQwNTMwLCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTczIiwic2NvcGUiOiJhY2NvdW50LnIgYWNjb3VudC53In0.M68F5O2ZlGpwJbxa91kOjRfNcbe0-_s6FEubPP1fjAp2MItZyyzkvnqMLrKlLv9ZbCiYiXKuBH1NDTOt93sDBzqlz7FeFutnxpUfNZhbg_dwhnVlWTvWmrQuFCILRDgKFlXRkLKcihZJI9OpjWMhno4WD5OmN6coyNRcoewhwS8Sg3UsGRobjSlKbc1Fo14_l6RaUdvX7AsPCC5J2uzajOG5a9oQiRVPJ1W4ecVPyYqdqBsWoUVZcsBLZcvnAagqzMBvoDQKmhlJ7WhmOw2fZxOeZSrrRtYBfdlC0xgdc6Lgi3R-W3ZdNAxhJ-Xypb06OpTR05FUuAJ639BIUo8_mQ'

```

##### Customer with right roles

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1952901318,
  "jti": "fxC4f3F5qmh9eg3zo4LFVA",
  "iat": 1637541318,
  "nbf": 1637541198,
  "version": "1.0",
  "user_id": "stevehu",
  "user_type": "CUSTOMER",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "roles": "customer",
  "scope": [
    "account.r",
    "account.w"
  ]
}
```


```
curl -k --location --request GET 'https://localhost:8080/accounts' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk1MjkwMTMxOCwianRpIjoiZnhDNGYzRjVxbWg5ZWczem80TEZWQSIsImlhdCI6MTYzNzU0MTMxOCwibmJmIjoxNjM3NTQxMTk4LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInJvbGVzIjoiY3VzdG9tZXIiLCJzY29wZSI6WyJhY2NvdW50LnIiLCJhY2NvdW50LnciXX0.bfG2okhBhgif2Jty60mGJKz2TKCtW219c2kcBVKznWctVmns8g0r0sztR_N2EBWZ-UUpA0Bm9kTo5DHoSGHM28t-46RSH_RdaTNGsRg74zLC_HJWuc6hGQl05jU-vltNNFPQ3CA0__yRNEi1zLqICtbqvmlcl0uHd_PnPeFvjNDRY68Qyr7PN_YXYbVT7dRiauqrWsslLZKbY0-Bpk8Ro6pJ03akX0-3pdd1Jy9HryyEPFw4OEwCqU2G_OETcZ2qNf-fKZwYLC9kofku9CehWkYhujpnuaFSOuGEGfB-eqi4tTHKA2YmaE-GsYUyFNa8H4cHTAGlKUmDRKRdV-em5Q'
```


##### Customer with wrong role


```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1952901422,
  "jti": "6M5kn6Ky0aE73kHRGIfpcw",
  "iat": 1637541422,
  "nbf": 1637541302,
  "version": "1.0",
  "user_id": "stevehu",
  "user_type": "CUSTOMER",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "roles": "user",
  "scope": [
    "account.r",
    "account.w"
  ]
}
```


```
curl -k --location --request GET 'https://localhost:8080/accounts' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk1MjkwMTQyMiwianRpIjoiNk01a242S3kwYUU3M2tIUkdJZnBjdyIsImlhdCI6MTYzNzU0MTQyMiwibmJmIjoxNjM3NTQxMzAyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInJvbGVzIjoidXNlciIsInNjb3BlIjpbImFjY291bnQuciIsImFjY291bnQudyJdfQ.VqExehutVEmo7qCuffCJZAPBWvoqihTWBuTBiXbR9x9vQbzPiFI0qr0FDilTxdqSOtEmW3ml9eioR2tLswLVp0NQnuHw5ElPYNfTsvIW8xLm3hcTAMk08Xhpg7TJn6_Z3zDNwv3mDn-ZMzB9R80O-OI61W2XAuWzCdOIEffcMTZa6VMB3e0tKLN3SnmKL5LJmbAxfuy8CK1QwfRLvhZgNYggd1XAyKCEB33VDEV0rKUJlwSRKXYZbKcvT1r1MojtP8JlReW9h_Xfx3CRH4VxzcAuVQyVrLd7bOpB03eVSkOTw9I4dgCe6ODELERrvGlsQ9aiETIn7rCrRsN9dt5mAw'
```

##### Employee with right group

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1952901546,
  "jti": "PUAAnkUgewwlJJC2LB1Cow",
  "iat": 1637541546,
  "nbf": 1637541426,
  "version": "1.0",
  "user_id": "stevehu",
  "user_type": "EMPLOYEE",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "groups": "admin frontOffice",
  "scope": [
    "account.r",
    "account.w"
  ]
}
```


```
curl -k --location --request GET 'https://localhost:8080/accounts' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk1MjkwMTU0NiwianRpIjoiUFVBQW5rVWdld3dsSkpDMkxCMUNvdyIsImlhdCI6MTYzNzU0MTU0NiwibmJmIjoxNjM3NTQxNDI2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJFTVBMT1lFRSIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsImdyb3VwcyI6ImFkbWluIGZyb250T2ZmaWNlIiwic2NvcGUiOlsiYWNjb3VudC5yIiwiYWNjb3VudC53Il19.nW6tu-L1qwnNMWdoEX-iAE04nlYB4rNXFYHtVS6aTBV1cwnRQZj7UygwrroOBRaRrsJKXMXkpXJ9MDfjGSurbMKZIZ-4iwqj2MK1xNnjSMIHj1hM0llNKtvjFCTlc-XJYFmvNbp6SW5YK47I3FVSRLNFEKopx75NpQu-hG_ysNbAcAoFXS8JA7bdb9SHxlbhbELBbbT7RB7GvifrU4_rvYD6PDAtRcHUOZtNBM1QbHSMSUa26P7mc7GinIC_zLJYHVblieNWvBzGdkhjhe5CQaE5mrjJvjJZUozfjg85hhRK4p_JkHz9urD9RDNnGF0u9LL1wR1QYK8USQiui-TVOw'
```


##### Employee with wrong group

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1952902670,
  "jti": "rqWkoHUxa-UnL6WDbPnJ5A",
  "iat": 1637542670,
  "nbf": 1637542550,
  "version": "1.0",
  "user_id": "stevehu",
  "user_type": "EMPLOYEE",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "groups": "backOffice",
  "scope": [
    "account.r",
    "account.w"
  ]
}
```

```
curl -k --location --request GET 'https://localhost:8080/accounts' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk1MjkwMjY3MCwianRpIjoicnFXa29IVXhhLVVuTDZXRGJQbko1QSIsImlhdCI6MTYzNzU0MjY3MCwibmJmIjoxNjM3NTQyNTUwLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJFTVBMT1lFRSIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsImdyb3VwcyI6ImJhY2tPZmZpY2UiLCJzY29wZSI6WyJhY2NvdW50LnIiLCJhY2NvdW50LnciXX0.UOi8a6bAzOnbIsFYlOZ9wvkKZbSZ8CHZg3VgGNZ_e287K-lWROMRIzfJOvud0IC6dWH8svIhME-c7lo6bL-4qd2juEMzIzbUSPYp7CX8iSpa1HEu6gYmdP6iSENQz9DwG9wxUwRwHZZOEaNubppdPGPUSIDW-Xz1PwslfgIyUHnVwhPjEpwlVPlQmEKr4in5N7EOUmpe8_MIo6brPBERhqtdljQr0luB9hafY0-ErYWqZDpZmbr8VxEx2kx4AItBkKi4GtYUiIUOum3SrAFZKz8CbBEKWtT_h6GPcI6NHWJGNOnpBbFyy0rG66_-EDo3-Br7VjNJqzt2Gg3dbNO50A'

```
