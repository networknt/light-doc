---
title: "Light Reference Router Config"
date: 2019-12-02T00:03:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have [deployed][] the reference API with docker-compose. However, the instance on test2 is not exposed to the Internet for security reasons. It has to be accessed from the light-portal server with virtual host configuration in the light-router instance. The light-router can enable security and all other tracing, metrics, etc. Also, it can host the single-page applications so that there is no need to wire in the CORS handler. 

For the test cloud environment, the light-router is deployed to the portal VM, and the configuration files can be found at [light-config-test](https://github.com/networknt/light-config-test/tree/master/light-router/test-portal) repository. 


Let's add `/rdata/{host}` endpoint to the handler.yml in the test-portal/config folder. 

handler.yml

```
  - path: '/rdata/{host}'
    method: 'GET'
    exec:
      - noauth
```

We need to update the pathPrefixService.yml file to add path prefix /rdata and service mapping. 

pathPrefixService.yml
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

```


check the handler.yml and pathPrefixService.yml in and open a session to the portal server. 

```
ssh portal
cd ~/networknt/light-config-test
git pull origin master
cd light-router/test-portal
docker-compose down
docker-compose up -d
```

Now, let's issue a curl command to see if the API is accessed from the Internet. 

```
curl https://lightapi.net/rdata/com.networknt?name=role
[{"admin":"admin"},{"user":"user"}]
```

The result means the API server is working. In the next step, we are going to update the [example][] of react-schema-form-rc-select to leverage this server for some dropdowns. 

[deploy]: /tutorial/rest/openapi/light-reference/docker-deploy/
[example]: /tutorial/rest/openapi/light-reference/rcselect-example/
