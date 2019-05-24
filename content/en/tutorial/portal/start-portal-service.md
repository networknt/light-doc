---
title: "Start Portal Service"
date: 2018-03-31T08:54:43-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---


Before starting light-portal services, the light-portal and its related repositories need to be built. Please follow the instructions in [build portal][] tutorial. 

Once light-portal is built, you can start command side and query side services with a docker-compose command. 

The docker-compose-cloud.yml is located at `~/networknt/light-config-test/light-portal` folder. 

docker-compose-cloud.yml

```
version: "2"
#
# Services
#
services:

  hybrid-command:
      build: ./hybrid-command/
      volumes:
        - ./hybrid-command/cloud/config:/config
        - ./hybrid-command/service:/service
      ports:
        - 8443:8443
      networks:
          - localnet

  hybrid-query:
      build: ./hybrid-query/
      volumes:
        - ./hybrid-query/cloud/config:/config
        - ./hybrid-query/service:/service
      ports:
        - 8442:8442
      networks:
          - localnet


# Networks
#
networks:
  localnet:
    # driver: bridge
    external: true
```

As you can see there are two services are included. Both read/write corresponding config files and service jar files with volume mappings. 

The hybrid-command service listens to port 8443, and hybrid-query service listens to port 8442.  

You can browse the config folders for each service. They are using the cloud testing eventuate event store so that there is no need to start eventuate services locally. 

To start the docker-compose: 

```
cd ~/networknt/light-config-test/light-portal
docker-compose -f docker-compose-cloud.yml up
```

Once the services are up and running, you can follow the [host-menu tutorial][] to confirm the portal services are working. 


[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/develop-build/build-portal
[build portal]: /tutorial/bot/light-portal-local/
[host-menu tutorial]: /tutorial/portal/host-menu/

