---
title: "Docker Compose"
date: 2017-11-29T05:40:14-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

So far, we have four servers running and they can talk to each other as standalone
services. In this step, we are going to dockerize all of them and start them with a
docker-compose up command. 

Before doing that, let's create two folders under ms_chain for different version of
compose files and externalized config files.

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain
mkdir compose
mkdir etc
```

Now in the etc folder, we need to create sub folders for each api and config sub folder


```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/etc
mkdir api_a
mkdir api_b
mkdir api_c
mkdir api_d

cd api_a
mkdir config
cd ../api_b
mkdir config
cd ../api_c
mkdir config
cd ../api_d
mkdir config
```


Let's create a rest/swagger/ms_chain/compose/docker-compose-app.yml.

```
#
# docker-compose-app.yml
#

version: '2'

#
# Services
#
services:


    #
    # Microservice: API A
    #
    apia:
        build: ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/metrics/
        ports:
            - "7441:7441"
        networks:
            - localnet
        volumes:
            - ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_a/config:/config

    #
    # Microservice: API B
    #
    apib:
        build: ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b/metrics/
        ports:
            - "7002:7002"
        networks:
            - localnet
        volumes:
            - ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_b/config:/config

    #
    # Microservice: API C
    #
    apic:
        build: ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/metrics/
        ports:
            - "7003:7003"
        networks:
            - localnet
        volumes:
            - ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_c/config:/config

    #
    # Microservice: API D
    #
    apid:
        build: ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d/metrics/
        ports:
            - "7004:7004"
        networks:
            - localnet
        volumes:
            - ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_d/config:/config

#
# Networks
#
networks:
    localnet:
        external: true
        
```


From the above docker compose file you can see we have a volume for each api with externalized
configuration folder under ms_chain/etc/api_x/config. Since we are using docker compose
we cannot use localhost:port to call services and we have to use service name in 
docker-compose-app.yml for the hostname. To resolve the issue without rebuilding the services
we are going to externalize api_a.yml, api_b.yml and api_c.yml to their externalized config
folder.

Let's create api_a.yml in ms_chain/etc/api_a/config folder. 

```
api_b_host: https://apib:7442
api_b_path: /v1/data
```

Please note that apib is the name of the service in docker-compose-app.yml

Create api_b.yml in ms_chain/etc/api_b/config folder

```
api_c_host: https://apic:7443
api_c_path: /v1/data

```

Create api_c.yml in ms_chain/etc/api_c/config folder

```
api_d_host: https://apid:7444
api_d_path: /v1/data
```

As each API needs to access OAuth2 server to get access token, so we need to update
client.yml to point to the service name in docker-compose instead of localhost. 

Now, let's copy the client.yml from metrics folder for each API into ms_chain/etc/api_x/config
folder and update the url for OAuth2 server. 

```
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/metrics/src/main/resources/config/client.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_a/config
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b/metrics/src/main/resources/config/client.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_b/config
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/metrics/src/main/resources/config/client.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_c/config
```

The api_a client.yml in ms_chain/etc/api_a/config folder looks like this after modification. 
Notice that server host has been changed to oauth2-token and oauth2-key from localhost.

```
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
    server_url: https://oauth2-token:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: 732bba11-9989-49ae-b26e-a29ed5b3f27e
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://localhost:8080/authorization_code
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - api_b.r
      - api_b.w
    # the following section defines uri and parameters for client credentials grant type
    client_credentials:
      # token endpoint for client credentials grant
      uri: "/oauth2/token"
      # client_id for client credentials grant flow. client_secret is in secret.yml
      client_id: 732bba11-9989-49ae-b26e-a29ed5b3f27e
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - api_b.r
      - api_b.w
  # light-oauth2 key distribution endpoint configuration
  key:
    # key distribution server url
    server_url: https://oauth2-key:6886
    # the unique service id for key distribution service
    serviceId: com.networknt.oauth2-key-1.0.0
    # the path for the key distribution endpoint
    uri: "/oauth2/key"
    # client_id used to access key distribution service. It can be the same client_id with token service or not.
    client_id: 732bba11-9989-49ae-b26e-a29ed5b3f27e
```

Now in order to make the metrics works, we need to copy the metrics.yml to the config folder
and modify it for the influxdb hostname.

```
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_a/metrics/src/main/resources/config/metrics.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_a/config
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_b/metrics/src/main/resources/config/metrics.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_b/config
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_c/metrics/src/main/resources/config/metrics.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_c/config
cp ~/networknt/light-example-4j/rest/swagger/ms_chain/api_d/metrics/src/main/resources/config/metrics.yml ~/networknt/light-example-4j/rest/swagger/ms_chain/etc/api_d/config
```

The api_a metrics.yml in ms_chain/etc/api_a/config folder looks like this after modification. 

```
# Metrics handler configuration

# If metrics handler is enabled or not
enabled: true

# influxdb protocal can be http, https
influxdbProtocol: http
# influxdb hostname
influxdbHost: influxdb
# influxdb port number
influxdbPort: 8086
# influxdb database name
influxdbName: metrics
# influxdb user
influxdbUser: root
# influx db password
influxdbPass: root
# report and reset metrics in minutes.
reportInMinutes: 1

```


Now let's start the docker compose. 

```
cd ~/networknt/light-example-4j/rest/swagger/ms_chain/compose
docker-compose -f docker-compose-app.yml up
```

Let's test if the servers are working.

```
curl -k -H "Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgwNjIwMDY2MSwianRpIjoibmQtb2ZZbWRIY0JZTUlEYU50MUFudyIsImlhdCI6MTQ5MDg0MDY2MSwibmJmIjoxNDkwODQwNTQxLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6IlN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJhcGlfYS53IiwiYXBpX2IudyIsImFwaV9jLnciLCJhcGlfZC53Iiwic2VydmVyLmluZm8uciJdfQ.SPHICXRY4SuUvWf0NYtwUrQ2-N-NeYT3b4CvxbzNl7D7GL5CF91G3siECrRBVexe0smBHHeiP3bq65rnCVFtwlYYqH6ZS5P7-AFiNcLBzSI9-OhV8JSf5sv381nk2f41IE4av2YUlgY0_mcIDo24ItnuPCxj0l49CAaLb7b1SHZJBQJANJTeQj-wgFsEqwafA-2wH2gehtH8CmOuuYfWO5t5IehP-zJNVT66E4UTRfvvZaJIvNTEQBWPpaZeeK6e56SyBqaLOR7duqJZ8a2UQZRWsDdIVt2Y5jGXQu1gyenIvCQbYLS6iglg6Xaco9emnYFopd2i3psathuX367fvw" https://localhost:7441/v1/data

```

And we will have the result.

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]
```
