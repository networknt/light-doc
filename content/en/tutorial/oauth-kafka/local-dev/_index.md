---
title: "OAuth Kafka Local Dev"
date: 2020-08-21T22:25:48-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Both light-portal and oauth-kafka depend on Kafka, and we are using confluent local to start Kafka and its services. To start confluent locally, please refer to [portal debug with unit test](/tutorial/portal/developer/debug-unit-test/) in the Confluent Platform section.

The oauth-kafka unit tests depend on the light-portal user and market services for both query and command. So you have to start the light-portal locally first. If you want to test the portal-view together, the light-router needs to be started as well. 

To avoid starting Consul locally, we can use direct registry for service discovery. 

start confluent

```
cd ~/tool/confluent-5.5.0/bin
confluent local start
```

shutdonw confluent

```
confluent local stop
```

start light-portal local-direct

```
cd ~/light-chain/light-config-test/light-portal/local-direct
docker-compose up
```

start light-router local-direct

```
cd ~/light-chain/light-config-test/light-router/local-direct
docker-compose up
```

enable nat redirect from 443 to 8443 for light-router

```
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -o lo -j REDIRECT --to-port 8443
```

From this moment on, you can enable the test cases in oauth-kafka modules and run them against light-portal and light-router. 

If you want to work on the portal-view lcally with all the services locally, you can start the oauth-kafka with the following commands.

```
cd ~/light-chain/light-config-test/oauth-kafka/local-direct
docker-compose up
```

To avoid errors, we have ignored all the test classes in the oauth-kafka so that it can be built without any services available locally.


