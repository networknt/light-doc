---
title: "Docker Compose Open Banking Services"
date: 2019-12-14T16:40:19-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have enabled [security][] and dockerize the services. In this section, we are going to create a docker-compose to start all services together with externalized configuration. We also register the services to the Consul cluster in the test cloud so that the client can discover them. 

We are going to use another repository for the deployment configuration, and it can be accessed at https://github.com/open-banking/light-config-test

First, let's create a service folder and a docker-compose.yml file.

```
  version: '2'
  services:
    accounts:
      image: networknt/com.networknt.ob.accounts-3.1.2:latest
      volumes:
        - ./accounts:/config
      environment:
        - STATUS_HOST_IP=${DOCKER_HOST_IP}
      network_mode: host    
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-token"
      #    env: "dev"
    balances:
      image: networknt/com.networknt.ob.balances-3.1.2:latest
      volumes:
        - ./balances:/config
      environment:
        - STATUS_HOST_IP=${DOCKER_HOST_IP}
      network_mode: host    
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-token"
      #    env: "dev"
    parties:
      image: networknt/com.networknt.ob.parties-3.1.2:latest
      volumes:
        - ./parties:/config
      environment:
        - STATUS_HOST_IP=${DOCKER_HOST_IP}
      network_mode: host    
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-service"
      #    env: "dev"
    transactions:
      image: networknt/com.networknt.ob.transactions-3.1.2:latest
      volumes:
        - ./transactions:/config
      environment:
        - STATUS_HOST_IP=${DOCKER_HOST_IP}
      network_mode: host    
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-client"
      #    env: "dev"
```

As you can see, each service has its config folder named as the service. 

Create these folders and copy the values.yml from accounts/security/src/main/resources/config. 

We also need to update/overwrite some of the values in the server.yml to enable registry and dynamic port. 

Let's update the values.yml as following. 

```
openapi-security.enableVerifyJwt: true
server.dynamicPort: true
server.enableRegistry: true
client.verifyHostname: false
```

To connect to the test cloud Consul cluster, we need to copy the `client.truststore` that contains the Consul certificate. We also need to copy the service.yml and consul.yml for Consul connectivity from https://github.com/networknt/light-config-test/tree/master/light-router/test-portal/config

We also need to copy the logback.xml to the config folder so that it gives us a chance to overwrite the default logging level if necessary. 

Checkin the light-config-test and clone it into the test2 server to start the compose.

```
ssh test2
mkdir open-banking
cd open-banking
git clone git@github.com:open-banking/light-config-test.git
cd light-config-test
cd service
docker-compose up -d
```

After the servers are started, you can see the console log as following. 

```
balances_1      | HOST IP 38.113.162.52
parties_1       | HOST IP 38.113.162.52
accounts_1      | HOST IP 38.113.162.52
transactions_1  | HOST IP 38.113.162.52
balances_1      | Https Server started on ip:0.0.0.0 Port:2414
parties_1       | Https Server started on ip:0.0.0.0 Port:2423
accounts_1      | Https Server started on ip:0.0.0.0 Port:2444
transactions_1  | Https Server started on ip:0.0.0.0 Port:2431
```

If you go to the Consul UI, you can find these services registered. You need the consul token in the consul.yml to access the consul UI at https://198.55.49.188:8500

Although the services are up and running with these ports, you cannot access them from the Internet as these ports are not opened in the firewall. Also, we don't want end-users to access these services with these dynamic ports. 

In the next step, we are going to set up a [router][] instance on the portal server to route all the requests to these services. 

{{< youtube OSpd85Y78_I >}}

[security]: /tutorial/open-banking/security/
[router]: /tutorial/open-banking/router/
