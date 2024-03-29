---
title: "Mras"
date: 2022-05-10T13:40:49-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

This is a customized handler that is used to access an external service through a corporate network with web gateway. The reason we cannot use the generic external service handler is that the service requires two-way TLS and customized access token endpoint. 

This handler is located in the light-4j repo ingress-proxy module. The default config template can be customized in the values.yml file. 

mras.yml

```
# Configuration for MrasHandler
# Indicate if the handler is enabled or not
enabled: ${mras.enabled:false}
# keyStoreName
keyStoreName: ${mras.keyStoreName:}
# keyStorePass
keyStorePass: ${mras.keyStorePass:}
# keyPass
keyPass: ${mras.keyPass:}
# trustStoreName
trustStoreName: ${mras.trustStoreName:apigatewayuat.pfx}
# trustStorePass
trustStorePass: ${mras.trustStorePass:password}
# Proxy Host if calling within the corp network with a gateway like Mcafee gateway.
proxyHost: ${mras.proxyHost:}
# Proxy Port if proxy host is used. default value will be 443 which means HTTPS.
proxyPort: ${mras.proxyPort:}
# If HTTP2 is used to connect to the MRAS site.
enableHttp2: ${mras.enableHttp2:false}
# When MrasHandler is used in the http-sidecar or light-gateway, it can collect the metrics info for
# the total response time of the downstream API. With this value injected, users can quickly determine how much
# time the http-sidecar or light-gateway handlers spend and how much time the downstream API spends, including
# the network latency. By default, it is false, and metrics will not be collected and injected into the metrics
# handler configured in the request/response chain.
metricsInjection: ${mras.metricsInjection:false}
# When the metrics info is injected into the metrics handler, we need to pass a metric name to it so that the
# metrics info can be categorized in a tree structure under the name. By default, it is mras-response, and
# users can change it.
metricsName: ${mras.metricsName:mras-response}

# URL rewrite rules, each line will have two parts: the regex patten and replace string separated
# with a space. For details, please refer to the light-router router.yml configuration.
# Test your rules at https://www.freeformatter.com/java-regex-tester.html
urlRewriteRules: ${mras.urlRewriteRules:}

# A list of applied request path prefixes and authentication config, other requests will skip this handler.
pathPrefixAuth: ${mras.pathPrefixAuth:}
# format with JSON for config server
#  {"/mras":"accessToken","/test":"basicAuth","/pdf/generator":"anonymous","/advisor":"microsoft"}

# format with YAML for readability
#  /mras: accessToken
#  /test: basicAuth
#  /pdf/generator: anonymous
#  /advisor: microsoft

# MRAS basic authentication configuration for some endpoints
basicAuth:
  # username for the authentication
  username: ${mras.basicAuth.username:}
  # password for the authentication
  password: ${mras.basicAuth.password:}
  # host for the endpoint
  serviceHost: ${mras.basicAuth.serviceHost:}
# MRAS accessToken issued by the MRAS service
accessToken:
  # MRAS get token URL to send the request
  tokenUrl: ${mras.accessToken.tokenUrl:https://test.mras.com/services/oauth2/token}
  # username for the authentication
  username: ${mras.accessToken.username:}
  # password for the authentication
  password: ${mras.accessToken.password:}
  # is cache enabled for the token
  cacheEnabled: ${mras.accessToken.cacheEnabled:}
  # memory key
  memKey: ${mras.accessToken.memKey:}
  # grace period
  gracePeriod: ${mras.accessToken.gracePeriod:}
  # host for the endpoint
  serviceHost: ${mras.accessToken.serviceHost:}

# MRAS Microsoft token issued by Microsoft service
microsoft:
  # MRAS get token URL to send the request
  tokenUrl: ${mras.microsoft.tokenUrl:https://test.mras.com/services/oauth2/token}
  # username for the authentication
  clientId: ${mras.microsoft.clientId:}
  # password for the authentication
  clientSecret: ${mras.microsoft.clientSecret:}
  # resource for the authentication
  resource: ${mras.microsoft.resource:}
  # is cache enabled for the token
  cacheEnabled: ${mras.microsoft.cacheEnabled:}
  # memory key
  memKey: ${mras.microsoft.memKey:}
  # grace period
  gracePeriod: ${mras.microsoft.gracePeriod:}
  # host for the endpoint
  serviceHost: ${mras.microsoft.serviceHost:}

# MRAS anonymous access for some endpoints.
anonymous:
  # MRAS target service host for service access with the token get with above property.
  serviceHost: ${mras.anonymous.serviceHost:}

```

The handler has a customzied TLS context object that is used to establish the two-way TLS connect to the target server. Also, it supports the proxyHost and proxyPort configuration to access the external service via the corporate gateway. 


