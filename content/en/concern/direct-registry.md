---
title: "Direct Registry"
date: 2023-01-03T21:38:14-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

If you don't have the Consul or the Light Portal deployed for the service registry and discovery, you can use the built-in DirectRegistry as a temporary solution. It is the best way to switch to an enterprise solution later. 

There are two ways to define the serviceId to the hosts mapping. The first option is using the service.yml and the second option is using the direct-registry.yml. 

### service.yml

It is the original way to define the parameters in the URLImpl in the service.yml. The problem with this approach is that we cannot reload the configuration without shutdown the server. When using the router feature in the http-sidecar and the light-gateway, we should use the direct-registry.yml for the configuration to reload the configuration with config-reload API. 

##### Single URL

This is the most simple service to URL mapping—one serviceId map to exactly one service instance. 

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002
        com.networknt.apic-1.0.0: http://localhost:7003
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```
##### Multiple URLs

For one serviceId, there are multiple service instances available. In this case, we need to put two or more URLs into one string separated by a comma.

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002,http://localhost:7005
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

##### With Tag

To support [multi-tenancy](/design/multi-tenancy/), we sometimes need to pass in an environment tag to lookup services. In this case, we need to put the environment parameter into the registered URLs.

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.portal.command-1.0.0: https://localhost:8440?environment=0000,https://localhost:8441?environment=0001,https://localhost:8442?environment=0002
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```

##### Summary

With the above examples, users can use the DirectRegistry for development or production. Here is an example of several services. 

service.yml

```
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
      parameters:
        com.networknt.apib-1.0.0: http://localhost:7002,http://localhost:7005
        com.networknt.apic-1.0.0: http://localhost:7003
        com.networknt.portal.command-1.0.0: https://localhost:8440?environment=0000,https://localhost:8441?environment=0001,https://localhost:8442?environment=0002
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster

```


### direct-registry.yml

When using the DirectRegsitry in the http-sidecar and the light-gateway, it is recommended to use this configuration instead of service.yml to support the config-reload. 

By using this config file, you have to make sure that you don't have a parameters section in the URLImpl configuration in the service.yml

For example, here is the service.yml example to enable the direct-registry.yml to take the DirectRegistry configuration for the serviceId and hosts mapping. 

service.yml

```
---
# singleton service factory configuration
singletons:
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: https
      host: localhost
      port: 8080
      path: direct
#      parameters:
#        code: http://192.168.1.100:6881,http://192.168.1.101:6881
#        token: http://192.168.1.100:6882
#        com.networknt.test-1.0.0: http://localhost,https://localhost
#        command: https://192.168.1.142:8440?environment=0000,https://192.168.1.142:8441?environment=0001,https://192.168.1.142:8442?environment=0002
- com.networknt.registry.Registry:
  - com.networknt.registry.support.DirectRegistry

```

Here is the default direct-registry.yml in the module. 


```
# direct-registry.yml
# For light-gateway or http-sidecar that needs to reload configuration for the router hosts, you can define the
# service to hosts mapping in this configuration to overwrite the definition in the service.yml file as part of
# the parameters. This configuration will only be used if parameters in the service.yml for DirectRegistry is null.

# directUrls is the mapping between the serviceId to the hosts separated by comma. If environment tag is used, you
# can add it to the serviceId separated with a vertical bar |
directUrls: ${direct-registry.directUrls:}

# The following is in YAML format.
#  code: http://192.168.1.100:6881,http://192.168.1.101:6881
#  token: http://192.168.1.100:6882
#  com.networknt.test-1.0.0: http://localhost,https://localhost
#  command|0000: https://192.168.1.142:8440
#  command|0001: https://192.168.1.142:8441
#  command|0002: https://192.168.1.142:8442

# The following is in JSON string format.
direct-registry.directUrls: {"code":"http://192.168.1.100:6881,http://192.168.1.101:6881","token":"http://192.168.1.100:6882","com.networknt.test-1.0.0":"http://localhost,https://localhost","command|0000":"https://192.168.1.142:8440","command|0001":"https://192.168.1.142:8441","command|0002":"https://192.168.1.142:8442"}

# The following is in string map format.

```

There are three ways to define the directUrls in the config file direct-registry.yml or values.yml

##### YAML format

Here is an exmaple in values.yml in YAML format. 

```
# direct-registry.yml
direct-registry.directUrls:
  code: http://192.168.1.100:6881,http://192.168.1.101:6881
  token: http://192.168.1.100:6882
  com.networknt.test-1.0.0: http://localhost,https://localhost
  # use tag separated by vertical bar from the serviceId as the key
  command|0000: https://192.168.1.142:8440
  command|0001: https://192.168.1.142:8441
  command|0002: https://192.168.1.142:8442

```

##### JSON format

Here is an example in values.yml in JSON string format for config server. 

```
direct-registry.directUrls: {"code":"http://192.168.1.100:6881,http://192.168.1.101:6881","token":"http://192.168.1.100:6882","com.networknt.test-1.0.0":"http://localhost,https://localhost","command|0000":"https://192.168.1.142:8440","command|0001":"https://192.168.1.142:8441","command|0002":"https://192.168.1.142:8442"}
```


##### Map string format

Here is an example in values.yml in Map string format for config server. 

```
direct-registry.directUrls: code=http://192.168.1.100:6881,http://192.168.1.101:6881&token=http://192.168.1.100:6882&com.networknt.test-1.0.0=http://localhost,https://localhost&command|0000=https://192.168.1.142:8440&command|0001=https://192.168.1.142:8441&command|0002=https://192.168.1.142:8442
```

##### Reload

When the DirectRegistry is initialized during the server startup, it will register itself to the ModuleRegistry. The [config-reload](/concern/config-reload/) API can be called to trigger the reload from the local filesystem values.yml or load from the config server. This allows the light-gateway and http-sdiecar to change the configuration and trigger the reload from the Control Pane without shutting down the server in a shared environment. 


Please note that only direct-registry.yml will support the configuration reload. If you have the mapping in the service.yml, reload is not supported. 

Also, as the RouterHandler will cache its copy of the mapping for the service lookup, we need to reload the RouterHandler simultaneously when DirectRegistry is reloaded. 

Here is the command that should be used to reload the router configuration simultaneously. 

```
curl --location --request POST 'https://localhost:8443/adm/modules' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTk4NDUzMTMyOSwianRpIjoiY1JvN3hpVHdGQmhKNndIdmlvTzZ2USIsImlhdCI6MTY2OTE3MTMyOSwibmJmIjoxNjY5MTcxMjA5LCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny00ZWZiLWE1MmQtNGM1Nzg3NDIxZTczIiwic2NvcGUiOiJhZG1pbiJ9.BHdrIB5EEwI2mhryQa-xnScDYMNOX01-tcRExBK6EjkaiUc_yfVhHG0o2uz5qQuSiNbyFu4q0IPsNYRAet48ghFurHmI64_LjIRTTKSZgRYkzpmutFuG9vMY81RFc9RggtVPq_DjMi4LSPtZEfq3giMkXOJOEh7Ja3rkUMuxA7mcR_ih7Fc0IiIGET5VIgUz9urqyAt9mj2F9BtPEFDSpRace3hSW2sPL-4BB5KKPNK_0OghX_fS4N0Gu979KTsXgk-wYrlLmz6xTTJ2yaDDJene3GQUtBddN6iQHSslTT9kadhvRualhhyY0tAifQ2Q57z96QPBbMhuk6LEU3jsuQ' \
--header 'Content-Type: application/json' \
--data-raw '[
    "com.networknt.router.middleware.PathPrefixServiceHandler",
    "com.networknt.router.RouterHandler",
    "com.networknt.registry.support.DirectRegistry"
]
'
```

The above three handlers are related, and they should be reloaded at the same time. 

