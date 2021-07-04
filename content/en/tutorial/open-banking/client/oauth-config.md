---
title: "Oauth Config"
date: 2020-02-14T22:27:55-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before we hook up the light-oauth2 to complete the Authorization Code grant flow, we need to create two clients and two users in the database configuration. As we want to register the light-oauth2 services to the Consul for the light-router to look up, we need to make a separate configuration in the [light-config-test][] repository. 

### docker-compose

We use a [docker-compose.yml][] to start the services. 

If you look at the above docker-compose.yml file, you will realize the following. 

All services are using `network_mode: host` to simplify the connectivity between different services. 

We pass the local IP address 192.168.1.144 as STATUS_HOST_IP so that the services register to Consul with this IP. 

To make sure that only this docker-compose file contains the local IP address, we are passing a domain to IP mapping to the docker containers. 

```
      extra_hosts:
        - "oauth.lightapi.net:192.168.1.144"

```

If you are trying this tutorial, you need to replace 192.168.1.144 with your local IP address. 

### database

In the [create-mysql.sql][], we need to add the following users and clients. 

```
-- add open banking users
INSERT INTO user_profile(user_id, user_type, first_name, last_name, email, roles, password)
VALUES('stevehu', 'customer', 'Steve', 'Hu', 'stevehu@gmail.com', 'user', '1000:5b39342c202d37372c203132302c202d3132302c2034372c2032332c2034352c202d34342c202d31362c2034372c202d35392c202d35362c2039302c202d352c202d38322c202d32385d:949e6fcf9c4bb8a3d6a8c141a3a9182a572fb95fe8ccdc93b54ba53df8ef2e930f7b0348590df0d53f242ccceeae03aef6d273a34638b49c559ada110ec06992');
INSERT INTO user_profile(user_id, user_type, first_name, last_name, email, roles, password)
VALUES('ericbroda', 'customer', 'Eric', 'Broda', 'ericbroda@gmail.com', 'user', '1000:5b39342c202d37372c203132302c202d3132302c2034372c2032332c2034352c202d34342c202d31362c2034372c202d35392c202d35362c2039302c202d352c202d38322c202d32385d:949e6fcf9c4bb8a3d6a8c141a3a9182a572fb95fe8ccdc93b54ba53df8ef2e930f7b0348590df0d53f242ccceeae03aef6d273a34638b49c559ada110ec06992');

```

The above statements add two users: stevehu/123456 and ericbroda/123456

```
-- Open Banking SPA demo client registration to https://ob.lightapi.net/authorization
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e74', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'browser', 'Open Banking SPA Demo', 'Open Banking Single Page Application React Client', 'accounts', '{"c1": "361", "c2": "67"}', 'https://ob.lightapi.net/authorization', 'admin');
-- Open Banking SPA demo client registration to https://localhost:3000/authorization
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e75', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'browser', 'Open Banking SPA Demo', 'Open Banking Single Page Application React Client', 'accounts', '{"c1": "361", "c2": "67"}', 'https://localhost:3000/authorization', 'admin');
```

The above statements add two clients: one for redirect site https://ob.lightapi.net and another for redirect site https://localhost:3000 for the react server. 


### start

Given that you have the light-config-test repository checked out in your workspace `networknt`, run the following commands.

```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose down
docker-compose up -d
```

You can double-check from the consul UI at http://localhost:8500/ui/dc1/services



[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-oauth2/local-consul
[docker-compose.yml]: https://github.com/networknt/light-config-test/blob/master/light-oauth2/local-consul/docker-compose.yml
[create-mysql.sql]: https://github.com/networknt/light-config-test/blob/master/light-oauth2/local-consul/light-oauth2/mysql/create_mysql.sql
[consul.yml]: https://github.com/networknt/light-config-test/blob/master/light-oauth2/local-consul/light-oauth2/mysql/config/oauth2-client/consul.yml
[service.yml]: https://github.com/networknt/light-config-test/blob/master/light-oauth2/local-consul/light-oauth2/mysql/config/oauth2-client/service.yml

