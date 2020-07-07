---
title: "Bootstrap From Key Service"
date: 2017-12-11T10:47:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When you are planning to turn it on in security.yml, you must ensure that you have [light-oauth2][]
microservices up and running. Also, you need to ensure that your client.yml is configured to connect
to the key endpoint correctly. 

Once it is set up correctly, your service instance will automatically goes to light-oauth2 key service
to get public key certificate to verify JWT token according to the kid in the header. For more details
please refer to [security cross-cutting concern][].


### Start light-oauth2 services

An easy way to start light-oauth2 microservices are through docker-compose. There are three docker-compose
files in [light-docker][] repository to allow you to start light-oauth2 with Mysql, Postgres or Oracle.

First let's clone the light-docker to your working directory. Here we assume you have a working directory
called networknt under your user home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
```

To start light-oauth2 services with Mysql database. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up
```

It will take about 30 seconds to have all services up and running. 

To confirm it is running properly, let's issue a curl command to the client service.

```
curl -k https://localhost:6884/oauth2/client?page=0
```

And you can see that there is a petstore client define in the light-oauth2 for testing. Here is the
output. 

```
[{"clientId":"f7d42348-c647-4efb-a52d-4c5787421e72","clientSecret":null,"clientType":"public","clientProfile":"mobile","clientName":"PetStore Web Server","clientDesc":"PetStore Web Server that calls PetStore API","ownerId":"admin","scope":"petstore.r petstore.w","redirectUri":"http://localhost:8080/authorization","createDt":"2017-12-11","updateDt":null}]
```

In this tutorial, we are going to use petstore service, if you are working on another service, you need
to register your service as a client through [client service][]. Also there is an [client service tutorial][] 
you can follow.

Please note that if your service is not calling other services, you only need to register itself as
service not client. Only when you want to bootstrap from key service or leverage key service for key
distribution once light-oauth2 key is rotated. 

### Build and Test petstore API

As we've got petstore registered in light-oauth2 by default, we are going to use that service for this
tutorial. Let's first clone [light-example-4j][] repository to your working directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
```  

Now let's build and start the petstore server.

```
cd ~/networknt/light-example-4j/rest/swagger/petstore
mvn clean install exec:exec
```

By default, security is not enabled, so you can access the petstore with the following command line.

```
curl -k https://localhost:8443/v2/pet/111
```

You should see the response like this.

```json
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

### Update security.yml

Now, let's update the security.yml to enable JWT verification and bootstrapFromKeyService. To avoid
accidentally checking in unwanted code, I will make a copy of petstore project and update on that
copied project. As the copied project end with .bak, it won't be checked into light-example-4j which
has .gitignore file to skip any directory with .bak suffix. 

```
cd ~/networknt/light-example-4j/rest/swagger
cp -r petstore petstore.bak
``` 

Now let's update the security.yml as following.

```yaml
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

# If OAuth2 provider support http2 protocol. If using light-oauth2, set this to true.
oauthHttp2Support: true

# Enable JWT token cache to speed up verification. This will only verify expired time
# and skip the signature verification as it takes more CPU power and long time.
enableJwtCache: true

# If you are using light-oauth2, then you don't need to have oauth subfolder for public
# key certificate to verify JWT token, the key will be retrieved from key endpoint once
# the first token is arrived. Default to false for dev environment without oauth2 server
# or official environment that use other OAuth 2.0 providers.
bootstrapFromKeyService: true

```

Note that enableVerifyJwt is true and bootstrapFromKeyService is true in above security.yml

In order to ensure that the public key is loaded from the light-oauth2 key service, we will remove
the oauth folder that contains local keys in the src/main/resources/config folder. 


### Update client module

When we generate the petstore project from [light-codegen][], the [config.json][] is specified to
not include client module in pom.xml and not include client.yml as well as client.keystore and
client.truststore in src/main/resources/config folder. 

If you want, you can update the config.json with supportClient to true and regenerate the petstore. 

Now let's manually update the petstore to enable client module in order to communicate with
light-oauth2 key service. 

First let's copy the client.keystore and client.truststore into src/main/resources/config/tls folder.
You can find these files in [light-4j repository client module][]. The easy way is to clone this
repo and copy the file over to light-example-4j. We also copy the client.yml and modify it according
to our needs.


```
cd ~/networknt
git clone https://github.com/networknt/light-4j.git
cp light-4j/client/src/main/resources/config/tls/* light-example-4j/rest/swagger/petstore.bak/src/main/resources/config/tls
cp light-4j/client/src/main/resources/config/client.yml light-example-4j/rest/swagger/petstore.bak/src/main/resources/config/client.yml

``` 

The updated client.yml should looks like this.

```yaml
# This is the configuration file for Http2Client.
---
# Settings for TLS
tls:
  # if the server is using self-signed certificate, this need to be false. If true, you have to use CA signed certificate
  # or load truststore that contains the self-signed cretificate.
  verifyHostname: true
  # trust store contains certifictes that server needs. Enable if tls is used.
  loadTrustStore: true
  # trust store location can be specified here or system properties javax.net.ssl.trustStore and password javax.net.ssl.trustStorePassword
  trustStore: tls/client.truststore
  # key store contains client key and it should be loaded if two-way ssl is uesed.
  loadKeyStore: false
  # key store location
  keyStore: tls/client.keystore
# settings for OAuth2 server communication
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: https://localhost:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://localhost:8080/authorization_code
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w
  # light-oauth2 key distribution endpoint configuration
  key:
    # key distribution server url
    server_url: https://localhost:6886
    # the unique service id for key distribution service
    serviceId: com.networknt.oauth2-key-1.0.0
    # the path for the key distribution endpoint
    uri: "/oauth2/key"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: f7d42348-c647-4efb-a52d-4c5787421e72

```

You need to pay attention on the oauth/key section for the server_url, uri and client_id. Please note
that the client_id is the same as the one returned from query client service. 

Also, you need to ensure that secret.yml contains the client_secret for the client_id above. The secret
is not returned from client query as it is only available right after the client is created. Here the
client_secret I have remembered in secret.yml in keyClientSecret. 

```
# This file contains all the secrets for the server and client in order to manage and
# secure all of them in the same place. In Kubernetes, this file will be mapped to
# Secrets and all other config files will be mapped to mapConfig

---

# Sever section

# Key store password, the path of keystore is defined in server.yml
serverKeystorePass: password

# Key password, the key is in keystore
serverKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
serverTruststorePass: password


# Client section

# Key store password, the path of keystore is defined in server.yml
clientKeystorePass: password

# Key password, the key is in keystore
clientKeyPass: password

# Trust store password, the path of truststore is defined in server.yml
clientTruststorePass: password

# Authorization code client secret for OAuth2 server
authorizationCodeClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Key distribution client secret for OAuth2 server
keyClientSecret: f6h1FTI8Q3-7UScPZDzfXA

# Consul service registry and discovery

# Consul Token for service registry and discovery
# consulToken: the_one_ring

```



### Build and Test again

Let's rebuilt and restart the server.

```
cd ~/networknt/light-example-4j/rest/swagger/petstore.bak
mvn clean install exec:exec
``` 

Now let's issue the same curl command as above and you will have an error as there is no token in
the header. 

```
curl -k https://localhost:8443/v2/pet/111
```

And the response.

```
{"statusCode":401,"code":"ERR10002","message":"MISSING_AUTH_TOKEN","description":"No Authorization header or the token is not bearer type"}
```

Let's include the long lived token for petstore in the request header. 

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA" https://localhost:8443/v2/pet/111
``` 

And the result should be

```json
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

### Summary

Above steps show you how to enable communication with light-oauth2 key service to get public key
certificate according to the kid header in JWT. Further, you can even enable bootstrapFromKeyService
to true so that you don't need to load public key certificates from config files. The key distribution
service is a unique feature of light-oauth2 to change the traditional push certificates to all server
instances to pull certificates from a central security server. 


[light-oauth2]: /service/oauth/
[security cross-cutting concern]: /concern/security/
[light-docker]: https://github.com/networknt/light-docker
[client service]: /service/oauth/service/client/
[client service tutorial]: /tutorial/oauth/client/
[light-example-4j]: https://github.com/networknt/light-example-4j
[light-codegen]: /tool/light-codegen/
[cofnig.json]: https://github.com/networknt/model-config/blob/master/rest/swagger/petstore/2.0.0/config.json
[light-4j repository client module]: https://github.com/networknt/light-4j/tree/master/client/src/main/resources/config/tls

