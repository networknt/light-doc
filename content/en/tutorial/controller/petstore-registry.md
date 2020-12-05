---
title: "Petstore Registry"
date: 2020-12-04T22:01:00-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this tutorial, we will start the light-controller with a docker-compose and start the petstore-regsitry from the light-example-4j to register it to the controller. Once we can see the instance on the controller dashboard, we can check the health, server info, and manipulate the petstore handler's logger level.

### Controller

To allow other services to register to the controller, we need to start the controller first.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-controler.yml up
```

### Petstore

Once the controller is up and running, we can start the petstore service.

```
cd ~/networknt/light-example-4j/rest/openapi/petstore-registry
mvn clean install exec:exec
```

After the server is running, a log entry will indicate the petstore has registered on the controller terminal.

### Dashboard

Go to the following URL, and you can see the controller dashboard.

```
https://localhost:8438/
```

On the dashboard, there should be one entry for the petstore service. Click on the expand button in front of serviceId; you can see the instance protocol, IP and port as well as three buttons. 

The Status Check button will give you the configuration for the health check, which is sent to the controller from the petstore during the registration phase. 

The Server Info button will give you the server info for the petstore service.

The Logger Config button will get all the pre-defined loggers with their levels for users to update them. On the result screen, users can click update to change the existing logger level or add a new logger for a package with a debugging and tracing level.

### Video

Here is a video walkthrough. 

{{< youtube m53r8yn4QtE >}}

