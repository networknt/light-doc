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

Make sure that 443 is accepted on the portal server. 

```
sudo ufw allow https
```

Make sure that 8443 is acceted on the portal server.

```
sudo iptables -I INPUT -p tcp -m tcp --dport 8443 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```

Port forwarding from 443 to 8443

```
sudo iptables -A PREROUTING -t nat -i ens2 -p tcp --dport 443 -j REDIRECT --to-port 8443
```

- Check Rules

After you change something, you might wondering what is the rules at the moment. You can export the current rules to a text file to check it out. And you can restore the rules with the text file. 

```
sudo iptables-save > iptables_rules.txt
```

You can use command line to check the rules. 

```
sudo iptables -L -n
```

As PREROUTING rule is part of the NAT, the above command line won't show the PREROUTING rule, to see them, use the following command. 

```
sudo iptables -L -n -t nat
```


- Remove Rules

If you make a mistake and add/insert an incorrect PREROUTING rule, you can delete it with the following commands. 

First you need to find out which line number the rule is. 

```
sudo iptables -t nat --line-numbers -L
```

Then delete the rule in PREROUTING by line number. 

```
sudo iptables -t nat -D PREROUTING 5
```

As the line number might be changed after removing one record, you need to rerun the --line-numbers command if you want to remove another record. 

