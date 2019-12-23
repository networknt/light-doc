---
title: "Open Banking Router"
date: 2019-12-14T20:06:06-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous step, we have created a [docker-compose][] to start all the services on the test2 server in the test cloud. In this step, we are going to add configuration to the light-portal router instance to enable routing requests to these services from the Internet. 

We have already got the router instance running on the portal server, and the configuration can be found at https://github.com/networknt/light-config-test/tree/master/light-router/test-portal

For this configuration, it is in the networknt organization, not the open-banking organization on the github.com. 

First, we need to add the endpoints to the handler.yml for the router config. 

```
  - path: '/accounts'
    method: 'GET'
    exec:
      - noauth
  - path: '/accounts/{AccountId}'
    method: 'GET'
    exec:
      - noauth
  - path: '/balances/accounts/{AccountId}'
    method: 'GET'
    exec:
      - noauth
  - path: '/balances'
    method: 'GET'
    exec:
      - noauth
  - path: '/parties'
    method: 'GET'
    exec:
      - noauth
  - path: '/transactions/accounts/{AccountId}'
    method: 'GET'
    exec:
      - noauth
  - path: '/transactions'
    method: 'GET'
    exec:
      - noauth

```

Second, we need to add the mapping from the prefix to the serviceId in the pathPrefixService.yml

```
# Whether to enable PathPrefixServiceHandler
enabled: true

# mapping from request path prefixes to serviceIds
mapping:
  /faucet: io.taiji.faucet-1.0.0
  /api/webclient: io.taiji.server-1.0.0
  /oauth2/service: com.networknt.oauth2-service-1.0.0
  /oauth2/user: com.networknt.oauth2-user-1.0.0
  /oauth2/password: com.networknt.oauth2-user-1.0.0
  /oauth2/client: com.networknt.oauth2-client-1.0.0
  /oauth2/token: com.networknt.oauth2-token-1.0.0
  /oauth2/code: com.networknt.oauth2-code-1.0.0
  /oauth2/key: com.networknt.oauth2-key-1.0.0
  /rdata: com.networknt.reference-1.0.0
  /accounts: com.networknt.ob.accounts-3.1.2
  /balances: com.networknt.ob.balances-3.1.2
  /parties: com.networknt.ob.parties-3.1.2
  /transactions: com.networknt.ob.transactions-3.1.2

```

Check-in the above config files and pull from the portal server. Restart the router docker-compose to enable the routing for the open-banking services. 

```
ssh portal
cd networknt/light-config-test
git pull origin master
cd light-router/test-portal
docker-compose down
docker-compose up -d
```

In order to access the service with a subdomain, we can add a subdomain in the DNS server so that we can access the services with ob.lightapi.net. 

Let's access the accounts service from the desktop or any computer from the Internet with a long-lived JWT token. 

```
curl -k https://ob.lightapi.net/accounts \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'

```

And we got the right response. 

```
{"Data":{"Account":[{"AccountId":"22289","Status":"Enabled","StatusUpdateDateTime":"2019-01-01T06:06:06+00:00","Currency":"GBP","AccountType":"Personal","AccountSubType":"CurrentAccount","Nickname":"Steve's Saving"},{"AccountId":"31820","Status":"Enabled","StatusUpdateDateTime":"2018-01-01T06:06:06+00:00","Currency":"GBP","AccountType":"Personal","AccountSubType":"CurrentAccount","Nickname":"Steve's Checking"}]},"Links":{"Self":"https://api.alphabank.com/open-banking/v3.1/aisp/accounts/"},"Meta":{"TotalPages":1}}
```

Let's try a specific account. 

```
curl -k https://ob.lightapi.net/accounts/22289 \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ' \
  -H 'x-fapi-financial-id: OB'
```

You can see the result contains the balance from the balances service and the last transaction amount from the transactions service. 

```
{"Data":{"Account":[{"Currency":"GBP","AccountId":"22289","Balance":"1230.00","LastTransactionAmount":"10.00","AccountType":"Personal","AccountSubType":"CurrentAccount","Nickname":"Steve's Saving"}]},"Meta":{"TotalPages":1},"Links":{"Self":"https://ob.lightapi.net/accounts/22289"}}
```

At this moment, our services are exposed to the Internet with HTTPS/HTTP2 enabled, and OAuth2 protected. To access it, use the subdomain ob.lightapi.net with the default HTTPS port. 

In the next step, we are going to build a single page application [client][] to access the APIs and deploy it on the router instance on the portal server as a virtual host. In this setup, there is no need to enable CORS to access the APIs from the SPA. 


[docker-compose]: /tutorial/open-banking/docker-compose/
[client]: /tutorial/open-banking/client/
