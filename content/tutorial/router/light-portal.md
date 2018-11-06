---
title: "Light Portal Virtual Hosts"
date: 2018-11-05T19:35:53-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to walk through the light-portal/light-router configuration for our dev environment. The configuration files are open sourced at https://github.com/networknt/light-config-test/tree/master/light-router but will move to our internal git server for production configuration. 

This folder is copied from virtual-host and we just made several modifications.

### docker-compose.yml

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    networks:
    - localnet
    ports:
    - 8443:8443
    volumes:
    - ./config:/config
    - ./faucet/build:/faucet/build
    - ./lightapi/build:/lightapi/build
    - ./taiji/build:/taiji/build

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true
```

### virtual-host.yml

```
hosts:
  - domain: faucet.taiji.io
    path: /
    #base: /home/steve/networknt/light-config-test/light-router/light-portal/faucet/build
    base: /faucet/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: taiji.io
    path: /
    #base: /home/steve/networknt/light-config-test/light-router/virtual-host/taiji/build
    base: /taiji/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  - domain: lightapi.net
    path: /
    #base: /home/steve/networknt/light-config-test/light-router/virtual-host/lightapi/build
    base: /lightapi/build
    transferMinSize: 10245760
    directoryListingEnabled: false
```

### Firewall

Follow the [Port 443 tutorial][] to setup the firewall on the server. 


After the iptables rules update, the ufw status looks like this.

```
 sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
8500/tcp                   ALLOW       Anywhere                  
Anywhere                   ALLOW       198.55.49.187             
Anywhere                   ALLOW       198.55.49.186             
22/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
8443/tcp                   ALLOW       Anywhere                  
8500/tcp (v6)              ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
8443/tcp (v6)              ALLOW       Anywhere (v6)      
```

And the PREROUTING rules look like. 

```
sudo iptables -t nat --line-numbers -L
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 8443
2    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 8443
3    DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DOCKER     all  --  anywhere            !localhost/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    MASQUERADE  all  --  172.19.0.0/16        anywhere            
2    MASQUERADE  all  --  172.17.0.0/16        anywhere            
3    MASQUERADE  tcp  --  172.19.0.2           172.19.0.2           tcp dpt:8443

Chain DOCKER (2 references)
num  target     prot opt source               destination         
1    RETURN     all  --  anywhere             anywhere            
2    RETURN     all  --  anywhere             anywhere            
3    DNAT       tcp  --  anywhere             anywhere             tcp dpt:8443 to:172.19.0.2:8443

```

[Port 443 tutorial]: /tutorial/security/port443/
