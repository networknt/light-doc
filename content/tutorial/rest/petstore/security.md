---
title: "Security"
date: 2017-11-08T10:55:22-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 70
aliases: []
toc: false
draft: false
---

By default, the generated API has security turned off. You can turn on the JWT 
verification by updating src/main/resources/config/security.yml in petstore project.

Here is the default security.yml

```
# Security configuration in light framework.
---
# Enable JWT verification flag.
enableVerifyJwt: false

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: true

# User for test only. should be always be false on official environment.
enableMockJwt: false

# JWT signature public certificates. kid and certificate path mappings.
jwt:
  certificate:
    '100': oauth/primary.crt
    '101': oauth/secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false
```

And here is the updated security.yml to make enableVerifyJwt to true.

```
# Security configuration in light framework.
---
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: true

# User for test only. should be always be false on official environment.
enableMockJwt: false

# JWT signature public certificates. kid and certificate path mappings.
jwt:
  certificate:
    '100': oauth/primary.crt
    '101': oauth/secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false
```

The updated security.yml enabled JWT signature verification as well as scope
verification. 

Let's shutdown the server by issuing a CTRL+C on the terminal. And restart the server
after repackage it.

```
mvn package exec:exec
```

With the security enabled, the curl command won't work anymore. 

```
curl -k https://localhost:8443/v2/pet/111
```

Result:

```
{"statusCode":401,"code":"ERR10002","message":"MISSING_AUTH_TOKEN","description":"No Authorization header or the token is not bearer type"}
```

In order to access it, you have to provide the right JWT token. There is a long lived
token that can be found at [https://github.com/networknt/light-oauth2](https://github.com/networknt/light-oauth2)

Let's use that token in curl.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" https://localhost:8443/v2/pet/111
```

And we have the result:

```
{
                "photoUrls" : [ "aeiou" ],
                "name" : "doggie",
                "id" : 123456789,
                "category" : {
                  "name" : "aeiou",
                  "id" : 123456789
                },
                "tags" : [
                  {
                    "name" : "aeiou",
                    "id" : 123456789
                  }
                ],
                "status" : "aeiou"
              }
```

In the next step, we are going to dockerize the API and then enable additional features
with externalized configuration files. 