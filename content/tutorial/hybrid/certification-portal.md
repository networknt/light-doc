---
title: "Certification Portal"
date: 2018-03-04T14:14:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This tutorial shows the steps to add api-certification hybrid service into light-portal
hybrid-query server so that we can access api-certification service from this server
instance. 

There are two ways to add a service to a hybrid server: 

### Update hybrid-query pom.xml

This service need to communicate with other service's server info endpoint, so client module
must be included into the hybrid-query server. 

We need to ensure that client module is included in the dependencies in pom.xml file. It is
already there so nothing needs to be done here. 

```xml
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>client</artifactId>
            <version>${version.light-4j}</version>
        </dependency>
```

Also there must be an client.yml config file in the externalized config folder. It doesn't
exist now, so the client.yml is added to ~/networknt/light-config-test/light-portal/hybrid-query/cloud/config

Also the client.keystore and client.truststore file needs to be copied from https://github.com/networknt/light-4j/tree/master/client/src/main/resources/config


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


Now let's build the server.

```
cd ~/networknt/light-portal/hybrid-query
mvn clean install
```

### Add service jar into service folder

You just need to build the api-certification project with "mvn clean install" and then add 
the jar file into service folder at ~/networknt/light-config-test/light-portal/hybrid-query/service

```
cd ~/networknt/light-portal/api-certification
mvn clean install
```

Copy the service jar file.

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/service
cp ~/networknt/light-portal/api-certification/target/certification-1.0.0.jar .

```

### Start the server

To start the server

```
cd ~/networknt/light-config-test/light-portal/hybrid-query/cloud
java -Dlight-4j-config-dir=./config -Dlogback.configurationFile=./logback.xml -cp ~/networknt/light-portal/hybrid-query/target/hybrid-query-1.0.0.jar:../service/* com.networknt.server.Server
```

### Test the service

In order to test the api-certification service, we need to have a demo server to connect to.
Here swagger petstore service will be used as target scanned service. 

Let's start swagger petstore server. 

```
cd ~/networknt/light-example-4j/rest/swagger/petstore
mvn clean install exec:exec
```

Once the server is up and running, let's issue a request to hybrid-query server. 

```
curl -k -H "Content-Type: application/json" -X POST -d '{"host":"lightapi.net","service":"certification","action":"certifyEndpoint","version":"0.1.0","data":{"host":"https://localhost:8443","path":"/v2/server/info","environment":"prod"}}' https://localhost:8082/api/json
```

The result should be something like this.

```json
[{"id":"iss0001","description":"Security component is not enabled.","url":"https://api.networknt.com/certification/issue/iss0001"},{"id":"iss0003","description":"The TLS key pair are still the default ones for development.","url":"https://api.networknt.com/certification/issue/iss0003"},{"id":"iss0004","description":"The public key certification for JWT token signature verification is the default one.","url":"https://api.networknt.com/certification/issue/iss0004"}]
```
