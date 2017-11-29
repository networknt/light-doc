---
title: "Security"
date: 2017-11-29T16:08:59-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

So far, we've started four servers and tested them successfully with both Http and Https
connections; however, these servers are not protected by OAuth2 JWT tokens as it is turned off
by default in the generated code. Now let's turn on the security and check the performance with
JWT verification is on.

Before we turn on the security, we need to have [light-oauth2](https://github.com/networknt/light-oauth2)
server up and running so that these servers can get JWT token in real time.

When we enable security, the source code needs to be updated in order to leverage 
client module to get JWT token automatically. Let's prepare the environment. 

#### Prepare APIs

Since we are going to change the code, let's copy each service into a new folder
called security from httpschain. 

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_a
cp -r httpschain security
cd ~/networknt/light-example-4j/rest/ms_chain/api_b
cp -r httpschain security
cd ~/networknt/light-example-4j/rest/ms_chain/api_c
cp -r httpschain security
cd ~/networknt/light-example-4j/rest/ms_chain/api_d
cp -r httpschain security

```


#### Update Config

Now let's update security.yml to enable JWT verification and scope verification
for each service. This file is located at src/main/resources/config folder.

API A

old file

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

Update to 

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

Update the security.yml for api_b, api_c and api_d in config folder to ensure
that enableVerifyJwt is true and enableVerifyScope is true.


#### Start OAuth2 Services

The easiest way to run light-oauth2 services is through docker-compose. In the preparation
step, we have cloned light-docker repo. 

Let's start the light-oauth2 services from a docker compose.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-oauth2-mysql.yml up
```
Now the OAuth2 services are up and running. 

#### Register Client

Before we start integrate with OAuth2 services, we need to register clients for api_a,
api_b, api_c and api_d. This step should be done from light-portal for official environment.
After client registration, we need to remember the client_id and client_secret for each in
order to update client.yml for each service. Please note, API A, B and C are service and client
at the same time.

For more details on how to use the command line tool or script to access oauth2 services,
please see this [tutorial](https://networknt.github.io/light-oauth2/tutorials/enterprise/)

Register a client that calls api_a.

```
curl -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"mobile","clientName":"Consumer","clientDesc":"A client that calls API A","scope":"api_a.r api_a.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' http://localhost:6884/oauth2/client

```

The return value is something like this. You returned object should be different and you need to write down the clientId and clientSecret. 
```
{"clientId":"e45cb3f4-d029-4284-a42f-a408de5fbe8a","clientSecret":"mCTYQQ2bTySQ-r1TZqY4Hg","clientType":"public","clientProfile":"mobile","clientName":"Consumer","clientDesc":"A client that calls API A","ownerId":"admin","scope":"api_a.r api_a.w","redirectUri":"http://localhost:8080/authorization","createDt":"2017-08-30","updateDt":null}
```

To make it convenient, I have created a long lived token that can access api_a with api_a scopes. Here is token and you can verify that in jwt.io
 
```
eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgxMzAwNTAzNiwianRpIjoiN2daaHY2TS14UXpvVDhuNVMxODNodyIsImlhdCI6MTQ5NzY0NTAzNiwibmJmIjoxNDk3NjQ0OTE2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.FkFbPTRXZf045_7fBlEPQTn7rNoib54TYQeFzSjLmMkUjrfDsJZD6EnrsAquDpHt8GKQNqGbyPzgiNWAIYHgwPZvM-lHw_dv0KUKii3D0woaFBkqu4vYxqyImROBii0B38evxPAZVONWqUncL21592bFPHsxGCz5oHL2unLv-oIQklWxcILpMrSL_tf7nhXHSu1RkRhshxAiAHSSpBZnluu4-jqZdEFtc5U_YApToUrKkmI_An1op5-6rS_I-fMbSnSctUoDgg3RT4Zvw1HC-ZLJlXWRF5-FD4uQOAOgy_T7PI75pNiuh4wgOGgdIf48X-7-fDkEbla-cVLiuj3z4g
```


Register a client for api_a to call api_b

```
curl -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"service","clientName":"api_a","clientDesc":"API A service","scope":"api_b.r api_b.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' http://localhost:6884/oauth2/client

```

And the result is something like this.

```
{"clientId":"732bba11-9989-49ae-b26e-a29ed5b3f27e","clientSecret":"hp0x-__HQZasm2f0gPih_Q","clientType":"public","clientProfile":"service","clientName":"api_a","clientDesc":"API A service","ownerId":"admin","scope":"api_b.r api_b.w","redirectUri":"http://localhost:8080/authorization","createDt":"2017-08-30","updateDt":null}
```

Now we need to update client.yml and secret.yml with this clientId and clientSecret for
api_a. These config files are located in src/main/resources/config folder. 


Here is the client.yml for api_a. You can see that client_id and scope are updated based
on the result of the client registration. 


```
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
oauth:
  tokenRenewBeforeExpired: 600000
  expiredRefreshRetryDelay: 5000
  earlyRefreshRetryDelay: 30000
  # token server url. The default port number for token service is 6882.
  server_url: http://localhost:6882
  authorization_code:
    # token endpoint for authorization code grant
    uri: "/oauth2/token"
    # client_id for authorization code grant flow. client_secret is in secret.yml
    client_id: 732bba11-9989-49ae-b26e-a29ed5b3f27e
    redirect_uri: https://localhost:8080/authorization_code
    scope:
    - api_b.r
    - api_b.w
  client_credentials:
    # token endpoint for client credentials grant
    uri: "/oauth2/token"
    # client_id for client credentials grant flow. client_secret is in secret.yml
    client_id: 732bba11-9989-49ae-b26e-a29ed5b3f27e
    scope:
    - api_b.r
    - api_b.w
```

And here is the secret.yml for api_a and you can see that authorizationCodeClientSecret
and clientCredentialsClientSecret are updated.

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
authorizationCodeClientSecret: hp0x-__HQZasm2f0gPih_Q

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: hp0x-__HQZasm2f0gPih_Q


```


Register a client for api_b to call api_c

```
curl -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"service","clientName":"api_b","clientDesc":"API B service","scope":"api_c.r api_c.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' http://localhost:6884/oauth2/client

```

And the result is

```
{"clientId":"9ace3ee5-7c01-453d-a80b-3c1e0bed8c40","clientSecret":"tjJGnU6ySjecmTDIbxXuXg","clientType":"public","clientProfile":"service","clientName":"api_b","clientDesc":"API B service","ownerId":"admin","scope":"api_c.r api_c.w","redirectUri":"http://localhost:8080/authorization","createDt":"2017-08-30","updateDt":null}
```

Now let's update client.yml for the client_id and scope like api_a and update secrets for
secret.yml

client.yml

```
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
oauth:
  tokenRenewBeforeExpired: 600000
  expiredRefreshRetryDelay: 5000
  earlyRefreshRetryDelay: 30000
  # token server url. The default port number for token service is 6882.
  server_url: http://localhost:6882
  authorization_code:
    # token endpoint for authorization code grant
    uri: "/oauth2/token"
    # client_id for authorization code grant flow. client_secret is in secret.yml
    client_id: 9ace3ee5-7c01-453d-a80b-3c1e0bed8c40
    redirect_uri: https://localhost:8080/authorization_code
    scope:
    - api_c.r
    - api_c.w
  client_credentials:
    # token endpoint for client credentials grant
    uri: "/oauth2/token"
    # client_id for client credentials grant flow. client_secret is in secret.yml
    client_id: 9ace3ee5-7c01-453d-a80b-3c1e0bed8c40
    scope:
    - api_c.r
    - api_c.w
```

secret.yml

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
authorizationCodeClientSecret: tjJGnU6ySjecmTDIbxXuXg

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: tjJGnU6ySjecmTDIbxXuXg

```

Register a client for api_c to call api_d

```
curl -H "Content-Type: application/json" -X POST -d '{"clientType":"public","clientProfile":"service","clientName":"api_c","clientDesc":"API C service","scope":"api_d.r api_d.w","redirectUri": "http://localhost:8080/authorization","ownerId":"admin"}' http://localhost:6884/oauth2/client

```

And the result is

```
{"clientId":"294e2fa6-fb2e-4fec-9fed-e9fe1d7fc83f","clientSecret":"TuC8EC49RZWAkDIY7bSmmw","clientType":"public","clientProfile":"service","clientName":"api_c","clientDesc":"API C service","ownerId":"admin","scope":"api_d.r api_d.w","redirectUri":"http://localhost:8080/authorization","createDt":"2017-08-30","updateDt":null}
```

We need to update client.yml and secret.yml according to the result of the registration

client.yml

```
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
oauth:
  tokenRenewBeforeExpired: 600000
  expiredRefreshRetryDelay: 5000
  earlyRefreshRetryDelay: 30000
  # token server url. The default port number for token service is 6882.
  server_url: http://localhost:6882
  authorization_code:
    # token endpoint for authorization code grant
    uri: "/oauth2/token"
    # client_id for authorization code grant flow. client_secret is in secret.yml
    client_id: 294e2fa6-fb2e-4fec-9fed-e9fe1d7fc83f
    redirect_uri: https://localhost:8080/authorization_code
    scope:
    - api_d.r
    - api_d.w
  client_credentials:
    # token endpoint for client credentials grant
    uri: "/oauth2/token"
    # client_id for client credentials grant flow. client_secret is in secret.yml
    client_id: 294e2fa6-fb2e-4fec-9fed-e9fe1d7fc83f
    scope:
    - api_d.r
    - api_d.w
```

secret.yml

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
authorizationCodeClientSecret: TuC8EC49RZWAkDIY7bSmmw

# Client credentials client secret for OAuth2 server
clientCredentialsClientSecret: TuC8EC49RZWAkDIY7bSmmw

```

API D is not calling any other API so it is not a client and doesn't need to be registered. However,
in the previous step httpschain, we have updated the test cases for api_d to and here we have enabled
the security and these two tests will fail as there is no security token in the test request. Let's
update the test case to put long lived token in it. 

Here is the updated test class. 

```
package com.networknt.apid.handler;

import com.networknt.client.Http2Client;
import com.networknt.exception.ApiException;
import com.networknt.exception.ClientException;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.junit.Assert;
import org.junit.ClassRule;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.IoUtils;
import org.xnio.OptionMap;
import java.net.URI;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

import java.io.IOException;


public class DataGetHandlerTest {
    @ClassRule
    public static TestServer server = TestServer.getInstance();

    static final Logger logger = LoggerFactory.getLogger(DataGetHandlerTest.class);
    static final boolean enableHttp2 = server.getServerConfig().isEnableHttp2();
    static final boolean enableHttps = server.getServerConfig().isEnableHttps();
    static final int httpPort = server.getServerConfig().getHttpPort();
    static final int httpsPort = server.getServerConfig().getHttpsPort();
    static final String url = enableHttp2 || enableHttps ? "https://localhost:" + httpsPort : "http://localhost:" + httpPort;

    @Test
    public void testDataGetHandlerTest() throws ClientException, ApiException {
        final Http2Client client = Http2Client.getInstance();
        final CountDownLatch latch = new CountDownLatch(1);
        final ClientConnection connection;
        try {
            connection = client.connect(new URI(url), Http2Client.WORKER, Http2Client.SSL, Http2Client.POOL, enableHttp2 ? OptionMap.create(UndertowOptions.ENABLE_HTTP2, true): OptionMap.EMPTY).get();
        } catch (Exception e) {
            throw new ClientException(e);
        }
        final AtomicReference<ClientResponse> reference = new AtomicReference<>();
        try {
            ClientRequest request = new ClientRequest().setPath("/v1/data").setMethod(Methods.GET);
            request.getRequestHeaders().put(Headers.AUTHORIZATION, "Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgxMzAwNTAzNiwianRpIjoiN2daaHY2TS14UXpvVDhuNVMxODNodyIsImlhdCI6MTQ5NzY0NTAzNiwibmJmIjoxNDk3NjQ0OTE2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.FkFbPTRXZf045_7fBlEPQTn7rNoib54TYQeFzSjLmMkUjrfDsJZD6EnrsAquDpHt8GKQNqGbyPzgiNWAIYHgwPZvM-lHw_dv0KUKii3D0woaFBkqu4vYxqyImROBii0B38evxPAZVONWqUncL21592bFPHsxGCz5oHL2unLv-oIQklWxcILpMrSL_tf7nhXHSu1RkRhshxAiAHSSpBZnluu4-jqZdEFtc5U_YApToUrKkmI_An1op5-6rS_I-fMbSnSctUoDgg3RT4Zvw1HC-ZLJlXWRF5-FD4uQOAOgy_T7PI75pNiuh4wgOGgdIf48X-7-fDkEbla-cVLiuj3z4g");
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            
            latch.await();
        } catch (Exception e) {
            logger.error("Exception: ", e);
            throw new ClientException(e);
        } finally {
            IoUtils.safeClose(connection);
        }
        int statusCode = reference.get().getResponseCode();
        String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
        Assert.assertEquals(200, statusCode);
        Assert.assertNotNull(body);
    }
}

```


Now we need to rebuild and restart API A, B, C and D in four terminals. 

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_d/security
mvn clean install exec:exec

```

```
cd ~/networknt/light-example-4j/rest/ms_chain/api_c/security
mvn clean install exec:exec
```


```
cd ~/networknt/light-example-4j/rest/ms_chain/api_b/security
mvn clean install exec:exec
```


```
cd ~/networknt/light-example-4j/rest/ms_chain/api_a/security
mvn clean install exec:exec
```


Now let's test it with a JWT token for the https port on api_a. For this moment on, 
we are going to use https all the time as it is required if OAuth2 is enabled.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgxMzAwNTAzNiwianRpIjoiN2daaHY2TS14UXpvVDhuNVMxODNodyIsImlhdCI6MTQ5NzY0NTAzNiwibmJmIjoxNDk3NjQ0OTE2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.FkFbPTRXZf045_7fBlEPQTn7rNoib54TYQeFzSjLmMkUjrfDsJZD6EnrsAquDpHt8GKQNqGbyPzgiNWAIYHgwPZvM-lHw_dv0KUKii3D0woaFBkqu4vYxqyImROBii0B38evxPAZVONWqUncL21592bFPHsxGCz5oHL2unLv-oIQklWxcILpMrSL_tf7nhXHSu1RkRhshxAiAHSSpBZnluu4-jqZdEFtc5U_YApToUrKkmI_An1op5-6rS_I-fMbSnSctUoDgg3RT4Zvw1HC-ZLJlXWRF5-FD4uQOAOgy_T7PI75pNiuh4wgOGgdIf48X-7-fDkEbla-cVLiuj3z4g" https://localhost:7441/v1/data
```

And you should have the normal result. 

