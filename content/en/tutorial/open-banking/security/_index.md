---
title: "Enable Security"
date: 2019-12-13T23:08:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For the real open-banking application, each user should only be allowed to access his/her own account information. This is controlled by the access token he/she can get from the OAuth 2.0 authorization code grant flow. 

In the previous [hardcoded][] step, we have all four microservices implemented based on two userIds: `stevehu` and `ericbroda`. 

Before we have the single page UI application implemented to interact with the light-oauth2 to get the JWT token, we are going to use two long-lived tokens for testing. 


token for userId stevehu

```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ
```

From the jwt.io site, here is the decoded payload. 

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1891704826,
  "jti": "QkmdtExNwt3jziFJPmZaPA",
  "iat": 1576344826,
  "nbf": 1576344706,
  "version": "1.0",
  "user_id": "stevehu",
  "user_type": "CUSTOMER",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "scope": [
    "accounts"
  ]
}
```


token for userId ericbroda

```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDg5NCwianRpIjoiRXBxVS1nVXJjTnFLbFV4Uk5tUUxfQSIsImlhdCI6MTU3NjM0NDg5NCwibmJmIjoxNTc2MzQ0Nzc0LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6ImVyaWNicm9kYSIsInVzZXJfdHlwZSI6IkNVU1RPTUVSIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOlsiYWNjb3VudHMiXX0.jhHSqIoU6c_3ceLPAFS8sGxmWYXNyO7zbnQpVk0jxRC1fdu12lKV4IPUA5KHl7184bsMe6JfuQ33qtOkf_xx05kOz4zkeh7Lzn9VhVgf7SIhgTlZJXsw-J77Q_Z1VwfOK2BsKfYDy_ldlZ3Trb5G19rY3AhL_-llA3rHbCMGpeMGwPEAOa8mg_BA5hg2gfrX89mja_r16n5sse_iX3y6G-dTYhMaDPqJTKcRU5LPO-BTazKQcmKuqT9sByV-CZKEzgUq8HHZKhavqNTluJz-KZxERZcCOq5gmM3FFIGWQ38m3IbGw0GADmA7Nl2s_IATUcU3ThvZCde-om4B_Uw8EQ
```

From the jwt.io site, here is the decoded payload.

```
{
  "iss": "urn:com:networknt:oauth2:v1",
  "aud": "urn:com.networknt",
  "exp": 1891704894,
  "jti": "EpqU-gUrcNqKlUxRNmQL_A",
  "iat": 1576344894,
  "nbf": 1576344774,
  "version": "1.0",
  "user_id": "ericbroda",
  "user_type": "CUSTOMER",
  "client_id": "f7d42348-c647-4efb-a52d-4c5787421e72",
  "scope": [
    "accounts"
  ]
}
```

Now, let's create security folders and copy the code from hardcoded to them in four projects. Also, we are going to add values.yml in each project resources/config folder to overwrite some default configuration values. 

Here is the file that enables the security for JWT token verification. Create the file under src/main/resources/config folder. In the real production deployment, it can be externalized. 

values.yml

```
openapi-security.enableVerifyJwt: true
```

With the JWT token verification enabled, we can get the userId from the AuditInfo exchange attachment in each handler to remove the hard-coded userId. 


Now, let's rebuild and restart the accounts service and using the default access token to access the /accounts endpoint. 

```
curl -k https://localhost:8443/accounts \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA' \
  -H 'x-fapi-financial-id: OB'

```

We got the following response because the token doesn't have the `accounts` scope defined in the specification. 


```
{"statusCode":403,"code":"ERR10005","message":"AUTH_TOKEN_SCOPE_MISMATCH","description":"Scopes [write:pets, read:pets] in authorization token and specification scopes [accounts] are not matched","severity":"ERROR"}
```

Let's switch to the token for `stevehu` above, and we got the right response. 

```
curl -k https://localhost:8443/accounts \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'

```

The returned response is still hard-coded, and we need to change the handlers to retrieve the userId from the JWT token. 

```
        Map<String, Object> auditInfo = exchange.getAttachment(AttachmentConstants.AUDIT_INFO);
        String userId = (String)auditInfo.get(Constants.USER_ID_STRING);

```

To use a specific accountId.

```
curl -k https://localhost:8443/accounts/22289 \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'

```

Let's use the token for `ericbroda` for another test. 

```
curl -k https://localhost:8443/accounts \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDg5NCwianRpIjoiRXBxVS1nVXJjTnFLbFV4Uk5tUUxfQSIsImlhdCI6MTU3NjM0NDg5NCwibmJmIjoxNTc2MzQ0Nzc0LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6ImVyaWNicm9kYSIsInVzZXJfdHlwZSI6IkNVU1RPTUVSIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOlsiYWNjb3VudHMiXX0.jhHSqIoU6c_3ceLPAFS8sGxmWYXNyO7zbnQpVk0jxRC1fdu12lKV4IPUA5KHl7184bsMe6JfuQ33qtOkf_xx05kOz4zkeh7Lzn9VhVgf7SIhgTlZJXsw-J77Q_Z1VwfOK2BsKfYDy_ldlZ3Trb5G19rY3AhL_-llA3rHbCMGpeMGwPEAOa8mg_BA5hg2gfrX89mja_r16n5sse_iX3y6G-dTYhMaDPqJTKcRU5LPO-BTazKQcmKuqT9sByV-CZKEzgUq8HHZKhavqNTluJz-KZxERZcCOq5gmM3FFIGWQ38m3IbGw0GADmA7Nl2s_IATUcU3ThvZCde-om4B_Uw8EQ' \
  -H 'x-fapi-financial-id: OB'

```

To get a specific account. 

```
curl -k https://localhost:8443/accounts/42281 \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDg5NCwianRpIjoiRXBxVS1nVXJjTnFLbFV4Uk5tUUxfQSIsImlhdCI6MTU3NjM0NDg5NCwibmJmIjoxNTc2MzQ0Nzc0LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6ImVyaWNicm9kYSIsInVzZXJfdHlwZSI6IkNVU1RPTUVSIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOlsiYWNjb3VudHMiXX0.jhHSqIoU6c_3ceLPAFS8sGxmWYXNyO7zbnQpVk0jxRC1fdu12lKV4IPUA5KHl7184bsMe6JfuQ33qtOkf_xx05kOz4zkeh7Lzn9VhVgf7SIhgTlZJXsw-J77Q_Z1VwfOK2BsKfYDy_ldlZ3Trb5G19rY3AhL_-llA3rHbCMGpeMGwPEAOa8mg_BA5hg2gfrX89mja_r16n5sse_iX3y6G-dTYhMaDPqJTKcRU5LPO-BTazKQcmKuqT9sByV-CZKEzgUq8HHZKhavqNTluJz-KZxERZcCOq5gmM3FFIGWQ38m3IbGw0GADmA7Nl2s_IATUcU3ThvZCde-om4B_Uw8EQ' \
  -H 'x-fapi-financial-id: OB'

```

You can see that the account returned is Eric's Saving and Checking. 

```
{"Data":{"Account":[{"AccountId":"42281","Status":"Enabled","StatusUpdateDateTime":"2019-01-01T06:06:06+00:00","Currency":"GBP","AccountType":"Personal","AccountSubType":"CurrentAccount","Nickname":"Eric's Saving"},{"AccountId":"41221","Status":"Enabled","StatusUpdateDateTime":"2018-01-01T06:06:06+00:00","Currency":"GBP","AccountType":"Personal","AccountSubType":"CurrentAccount","Nickname":"Eric's Checking"}]},"Links":{"Self":"https://api.alphabank.com/open-banking/v3.1/aisp/accounts/"},"Meta":{"TotalPages":1}}
```

Let's make the same change in all handlers for accounts, balances, parties, and transactions. You can test it with the two tokens above for other services after that. 

To access balances, for example.

```
curl -k https://localhost:8443/balances/accounts/22289 \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'

```

Now, let's check in the security folder for all four projects.

At this moment, all four services are functioning. Let's build Docker images and publish them to the docker hub. 

```
cd ~/open-banking/accounts/security
chmod +x build.sh
./build.sh 1.0.0
```

Follow the above command lines to build and publish the other three services. 

In the next step, we are going to create a [docker-compose][] to start all services on the test cloud together. 

[hardcoded]: /tutorial/open-banking/hardcoded/
[docker-compose]: /tutorial/open-banking/docker-compose/
