---
title: "Consul Registry Checklist"
date: 2019-04-28T23:46:04-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When you are using Consul for service registry and discovery, the following configuration updates need to be done to connect to the consul cluster. 

* client.truststore and client.keystore

The Consul certificate needs to be imported into the client.truststore. 

* client.yml

If the consul cluster doesn't have a domain name, you need to set the verifyHostname to false

* consul.yml

update the consulUrl and other parameters. User httpCheck if possible.

* secret.yml

make sure that the consul token is in this file.

* server.yml

enableRegisty to true.  If dynamicPort is true, you need to specify minPort and MaxPort

* service.yml

add the section for consul registry

* docker-compose

make sure that host network is used and DOCKER_HOST_IP is passed in.

* handler.yml

If httpCheck is used, you may need to update the handler.yml to add health/{serviceId} endpoint. 

* If httpCheck is used, make sure that you have firewall open for the port

* If host network is used, all references in for the docker service need to be changed to a real IP address instead of the service name. 

