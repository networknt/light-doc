---
title: "Token Limit Handler"
date: 2024-10-11T11:49:16-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

For most OAuth 2.0 cloud service providers, customers are charged based on the number of tokens issued. When a JWT (JSON Web Token) is used, it typically expires within 10 to 20 minutes. Therefore, it's practical for the client to cache the token within the expiration period to avoid requesting a new token for each API call. This approach also allows the API server to cache the token instead of verifying the signature for every request, as the cryptographic calculation is time-consuming.


### OAuth Token Management Strategy for Large Organizations
In large organizations with thousands of applications and APIs, detecting inefficient token usage (such as requesting a new token per API call) is challenging. Several factors contribute to this difficulty:

* IP Address Detection: Accurately identifying source IP addresses is problematic due to numerous network components between clients and servers.
* Token Analysis: Each token must be parsed to extract claims, allowing detection of multiple token uses within the expiration period for the same client_id (in the case of client credentials grant type).
* Performance Impact: Implementing this detection before the token security verification handler would slow down all APIs within the organization.

Given these challenges, applying limits at the OAuth 2.0 provider's token endpoint seems ideal. However, most cloud OAuth 2.0 providers lack this feature, and standardization is difficult.

#####  Proposed Solution

To create a generic, standardized solution, we propose:

* Set up a light-gateway instance for all external and internal OAuth 2.0 providers.
* Route each token request through this light-gateway.
* Implement a light-4j handler to record all token requests.
* Use this data to identify applications that are inefficiently requesting new tokens for each API call.

This approach allows for centralized monitoring and management of token usage across the organization, regardless of the specific OAuth 2.0 provider in use.


##### Deployment Options

There are two options for the light-gateway setup:

* Dedicated Light-Gateway: A light-gateway specifically for OAuth 2.0 providers only.

* Pros:

Focused on token management
Potentially simpler configuration


* Cons:

Additional infrastructure to maintain




* Shared Outbound Light-Gateway: A single light-gateway for all external services, including OAuth 2.0 providers.

* Pros:

Consolidated infrastructure
Potential for broader traffic analysis


* Cons:

More complex configuration
May require more resources to handle diverse traffic


The choice between these options depends on factors such as organizational structure, existing infrastructure, and specific security requirements.


### Design

#### Detailed Design
The light-gateway handler for managing token requests is designed with the following specifications:

##### Handler Configuration

* The handler will be part of the default chain.
* It will match the request path with a list of configured templates for the token request paths, accommodating different paths used by various OAuth 2.0 providers.

##### Data Management

* Stateful data is cached in the CacheManager named "token-limit".
* Configuration for the CacheManager should be specified in the cache.yml section of the values.yml file.

##### Handler Positioning

* The handler must be positioned after the requestInterceptor in the chain, as it depends on the REQUEST_BODY_STRING attachment.

##### Request Differentiation

The handler differentiates between client_credentials and authorization_code grant types:

* For client_credentials: The key is composed of client_id and source IP.
* For authorization_code: The key is composed of client_id, code and source IP.
* Refresh token is one-time token and it is not handled in this handler.


##### Duplicate Request Handling

The maximum number of duplicated keys is configurable as duplicateLimit in the config file.
When duplicated request counts exceed this number, an action is recommended based on the environment:

* For lower environments (dev/sit/stg): It is recommended to return an error message to the consumer.
* For production environment: It is recommended to log the same error in the logs, but not return it to the consumer.

These recommendations allow for flexibility in implementation while providing guidance on best practices for different environments.


##### Implementation Considerations

* Flexibility: The design allows for easy adaptation to different OAuth 2.0 providers and organizational needs.
* Scalability: Using a CacheManager enables efficient handling of large volumes of requests.
* Security: Differentiating between grant types and using composite keys helps in precise tracking and limiting of token requests.
* Environment-specific behavior: The different handling for production vs. non-production environments allows for stricter control in lower environments while maintaining service availability in production.

This design ensures that the light-gateway can effectively monitor and manage token requests across various OAuth 2.0 providers, helping to identify and mitigate inefficient token usage patterns.


### Configuration

Here is the token-limit.yml config file with default values.

```
# indicate if the handler is enabled or not. By default, it is enabled.
enabled: ${token-limit.enabled:true}
# return an error if limit is reached. It should be the default behavior on dev/sit/stg. For production,
# a warning message should be logged. Also, this handler can be disabled on production for performance.
errorOnLimit: ${token-limit.errorOnLimit:true}
# The max number of duplicated token requests. Once this number is passed, the limit is triggered. This
# number is set based on the number of client instances as each instance might get its token if there
# is no distributed cache. The duplicated tokens are calculated based on the local in memory cache per
# light-gateway or oauth-kafka instance. Note: cache.yml needs to be configured.
duplicateLimit: ${token-limit.duplicateLimit:2}
# Different OAuth 2.0 providers have different token request path. To make sure that this handler only
# applied to the token endpoint, we define a list of path templates here to ensure request path is matched.
# The following is an example with two different OAuth 2.0 providers in values.yml file.
# token-limit.tokenPathTemplates:
#   - /oauth2/(?<instanceId>[^/]+)/v1/token
#   - /oauth2/(?<instanceId>[^/]+)/token
tokenPathTemplates: ${token-limit.tokenPathTemplates:}

```


Here is the values.yml file that configures a shared light-gateway instance to router traffic to Okta token endpoint. 

```
# request-injection.yml
request-injection.appliedBodyInjectionPathPrefixes:
  - /v1/flowers
  # This is the path prefix for the Okta token endpoint. We need to add it to inject the request body. 
  - /oauth2


# pathPrefixService.yml
pathPrefixService.mapping:
  # /gateway/Migration/1.0.0: Migration
  /v1/pets: com.networknt.petstore-1.0.0
  /v1/notifications: com.networknt.petstore-1.0.0
  /v1/flowers: com.networknt.petstore-1.0.0
  /v1/documents: com.networknt.petstore-1.0.0
  # This is the path prefix setup for router. It has a service name okta-oauth2
  /oauth2: okta-oauth2


# direct-registry.yml
direct-registry.directUrls:
  com.networknt.petstore-1.0.0: https://localhost:9443
  # Here okta-oauth2 service name is mapped to a Okta host. 
  okta-oauth2: https://networknt.oktapreview.com


# handler.yml
handler.basePath: /
handler.handlers:
.
.
.
  # register the TokenLimitHandler and give it an alias tokenLimit. 
  - com.networknt.token.limit.TokenLimitHandler@tokenLimit

handler.chains.default:
  - exception
.
.
.
  - requestInterceptor
  - responseInterceptor
  - audit
  # put the tokenLimit alias into the default chain after the requestInterceptor
  - tokenLimit

# body.yml
# Need to cache the request body so that we can get the string formated request body from the exchange.
body.cacheRequestBody: true

# security.yml
# Disable the security for /oauth2 so that you don't need a token to access the token endpoint. 
security.skipPathPrefixes: ["/v1/documents", "/v1/pets", "/adm/server/info", "/oauth2"]


# cache.yml
cache.caches:
  - cacheName: jwt
    expiryInMinutes: 15
    maxSize: 100
  - cacheName: jwk
    expiryInMinutes: 129600
    maxSize: 100
  # Add token-limit cache for TokenLimitHandler.
  - cacheName: token-limit
    expiryInMinutes: 15
    maxSize: 100

# token-limit.yml
token-limit.tokenPathTemplates:
  - /oauth2/(?<instanceId>[^/]+)/v1/token

```

### Test

Here is the curl command that access the local gateway. 

```
curl --location --request POST 'https://localhost:8443/oauth2/aus6wg8qoeo4eCbc71d7/v1/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=0oa6wgbkbcF27GoqA1d7' \
--data-urlencode 'client_secret=GInDco_MGt6Fz0oHxmgk1LluEHb6qJ4RQY0MvH3Q' \
--data-urlencode 'scope=petstore.w'
```

Here is the error response if limit is reached. 

```
{
    "statusCode": 400,
    "code": "ERR10091",
    "message": "TOKEN_LIMIT_ERROR",
    "description": "Too many token requests. Please cache the jwt token on the client side.",
    "severity": "ERROR"
}
```

### Tutorial

