---
title: "Service Config"
date: 2021-06-15T11:49:31-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

For any service built with the light-4j frameworks to leverage the light-config-server, we need to add some extra configuration files and environment variables to bootstrap the service. 

### startup.yml

Create a startup.yml file to enable the config loader for the a service to bootstrap with the light-config-server. 

startup.yml

```
# This is the config file to replace the default config loader to a customized one.
# For example, load the config files from the config server instead of filesystem.
# The following dummy entry is just to prevent the warning message during startup.
# dummy: dummyEntry
# For real config loader config, please follow the format below with your implementation.
configLoaderClass: com.networknt.server.DefaultConfigLoader
# Project name and version where this API belongs to
projectName: example
projectVersion: 0.0.1

# Service name and version
serviceName: example-service
serviceVersion: 0.0.1
```

### Environment Variables

Before starting the service, we need to set up the following environment variables. These are used to bootstrap the service, connect to the config server, load config files, cache loaded files, and start the service with the config files.

When using the Docker container, the following variables should be passed into the container.

```
export CONFIG_SERVER_CLIENT_TRUSTSTORE_LOCATION=/config/bootstrap.truststore
export CONFIG_SERVER_CLIENT_TRUSTSTORE_PASSWORD=password
export CONFIG_SERVER_CLIENT_VERIFY_HOST_NAME=false
export LIGHT_ENV=dev
export LIGHT_4J_CONFIG_DIR=/config
export LIGHT_CONFIG_SERVER_URI=https://localhost:8443
```

