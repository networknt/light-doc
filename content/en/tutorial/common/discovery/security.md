---
title: "Enable Security"
date: 2019-06-18T22:32:23-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In previous steps, we have explored all different options of service discovery. In this step, we are going to build on top of http-health codebase and enable the security with light-oauth2. It is requested by some users who want to see how different features work together in a distributed application. 

### Start Consul

```
docker run -d -p 8400:8400 -p 8500:8500/tcp -p 8600:53/udp -e 'CONSUL_LOCAL_CONFIG={"acl_datacenter":"dc1","acl_default_policy":"allow","acl_down_policy":"extend-cache","acl_master_token":"the_one_ring","bootstrap_expect":1,"datacenter":"dc1","data_dir":"/usr/local/bin/consul.d/data","server":true}' consul agent -server -ui -bind=127.0.0.1 -client=0.0.0.0

```

### Start light-oauth2

The default security for light-4j is OAuth 2.0 and light-oauth2 is an implementation of a set of microservices. It is very easy to start it with the docker-compose in light-docker repository. 

If you haven't checked out the light-docker, you can. 

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
```

Make sure you have Docker installed on your desktop or test server. Start the light-oauth2 with the following command. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up -d
```
### Client Registration

Since we have API A calls API B, API C and API B calls API D, we need to register API B to the light-oauth2 to get scope token to the next API D. When we call the API A, we can use the long-lived token. We can register the API B as a client by accessing the API from curl. For more information about light-oauth2, please check the [light-oauth2][] tutorial. 

API D needs scope ad.r or ad.w to access the endpoint, so we need to make the API B client registrion like the following. 

```
curl -k -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"mobile","clientName":"API B","clientDesc":"API B","scope":"ad.r ad.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' https://localhost:6884/oauth2/client
```

The return result contains the generated client_id and client_secret we need in the API B client.yml configuration later on. 


```
{"clientId":"2cf4d072-f53a-46e8-b710-037fabcba4f3","clientSecret":"sRFRUmjJRZ-Zpq15CC9u7g","clientType":"public","clientProfile":"mobile","clientName":"API B","clientDesc":"API B","ownerId":"admin","scope":"ad.r ad.w","customClaim":null,"redirectUri":"http://localhost:8080/authorization","authenticateClass":null,"derefClientId":null}

```

API A is calling API B and API C, so we need to register it on the light-oauth2 as a client. It needs to have access to scope ab.r and ac.r

```
curl -k -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"mobile","clientName":"API A","clientDesc":"API A","scope":"ab.r ac.r","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' https://localhost:6884/oauth2/client
```

The returned result will be used to config the client.yml for API A. 


```
{"clientId":"f10ab624-16b5-4355-8b7a-aa4b37351855","clientSecret":"kve4koSJR1S83ho4tcaOiQ","clientType":"public","clientProfile":"mobile","clientName":"API A","clientDesc":"API A","ownerId":"admin","scope":"ab.r ac.r","customClaim":null,"redirectUri":"http://localhost:8080/authorization","authenticateClass":null,"derefClientId":null}

```

### Create security folder

Let's copy from http-health to security for each API. 

```
cd ~/networknt/light-example-4j/discovery/api_a
cp -r http-health security
cd ~/networknt/light-example-4j/discovery/api_b
cp -r http-health security
cd ~/networknt/light-example-4j/discovery/api_c
cp -r http-health security
cd ~/networknt/light-example-4j/discovery/api_d
cp -r http-health security
```

### API D

After we copy the folders, we need to update these APIs. 

We need to change the consulUrl to point to the local consul server with http. 

consul.yml

```
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: true
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: false
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2, HTTP/2 used to be default but some users said that newer version of Consul doesn't support HTTP/2 anymore.
enableHttp2: false

```

Enable security

openapi-security.yml

```
enableVerifyJwt: true
```

Update server.yml to ensure that dynamicPort is false. 

```
dynamicPort: false
```

### API C

API C will be called by API A and it won't call any other API. We need to simply enable the security.

openapi-security.yml

```
# Security configuration for openapi-security in light-rest-4j. It is a specific config
# for OpenAPI framework security. It is introduced to support multiple frameworks in the
# same server instance. If this file cannot be found, the generic security.yml will be
# loaded for backward compatibility.
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
    '100': primary.crt
    '101': secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: false

```


update consulUrl and consulToken in consul.yml

```
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
```

Update server.yml to ensure that dynamicPort is false. 

```
dynamicPort: false
```


### API B

For API B, we need to update client.yml for the client_id and client_secret genreated in the previous step during the client registration of light-oauth2.

Here is the updated section in client.yml for the client_credentials grant flow. 

```
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: 2cf4d072-f53a-46e8-b710-037fabcba4f3
      # client_secret for client credentials grant flow.
      client_secret: sRFRUmjJRZ-Zpq15CC9u7g
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - ad.r
      - ad.w

```

Next we need to update the openapi-security.yml for API B to enable security.

openapi-security.yml


```
# Security configuration for openapi-security in light-rest-4j. It is a specific config
# for OpenAPI framework security. It is introduced to support multiple frameworks in the
# same server instance. If this file cannot be found, the generic security.yml will be
# loaded for backward compatibility.
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
    '100': primary.crt
    '101': secondary.crt
  clockSkewInSeconds: 60

# Enable or disable JWT token logging
logJwtToken: true

# Enable or disable client_id, user_id and scope logging.
logClientUserScope: false

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: false

```

update consulUrl and consulToken in consul.yml

```
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
```

Update server.yml to ensure that dynamicPort is false. 

```
dynamicPort: false
```

### API A

Update openapi-security to enable security first. In order to use the long-lived token to access API A with curl, we are going to disable the scope verfication in the openapi-security.yml

```
# Enable JWT verification flag.
enableVerifyJwt: true

# Enable JWT scope verification. Only valid when enableVerifyJwt is true.
enableVerifyScope: false

```

And then update the client.yml for the client_credentials section with the client_id and client_secret returned from light-oauth2 registration. 

```
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: f10ab624-16b5-4355-8b7a-aa4b37351855
      # client_secret for client credentials grant flow.
      client_secret: kve4koSJR1S83ho4tcaOiQ
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - ab.r
      - ac.r

```

update consulUrl and consulToken in consul.yml

```
# Consul URL for accessing APIs
consulUrl: http://localhost:8500
# access token to the consul server
consulToken: the_one_ring
```

Update server.yml to ensure that dynamicPort is false. 

```
dynamicPort: false
```

## Starting the servers

Now let's start four terminals to start servers.  

**API A**

```bash
cd ~/networknt/light-example-4j/discovery/api_a/security
mvn clean install -Prelease
java -jar target/aa-1.0.0.jar
```

**API B**

```bash
cd ~/networknt/light-example-4j/discovery/api_b/security
mvn clean install -Prelease
java -jar target/ab-1.0.0.jar
```

**API C**

```bash
cd ~/networknt/light-example-4j/discovery/api_c/security
mvn clean install -Prelease
java -jar target/ac-1.0.0.jar
```

**API D**

And start the first instance that listen to 7445 as default

```bash
cd ~/networknt/light-example-4j/discovery/api_d/security
mvn clean install -Prelease
java -jar target/ad-1.0.0.jar
```

### Test

We can test the services with the following curl command. 

```
curl -k https://localhost:7441/v1/data \
  -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA' \
  -H 'Content-Type: application/json'
```

And the following result is expected. 

```
["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2","API C: Message 1","API C: Message 2","API A: Message 1","API A: Message 2"]
```


With the security enabled, the client module is propagating the original token, traceabilityId and correlationId to the entire chain of the call tree. In the next tutorial, we are going to replace the traceability and correlation handlers with [OpenTracing][]. 

[light-oauth2]: /tutorial/oauth/
[OpenTracing]: /tutorial/tracing/jaeger/service-discovery/
