---
title: "Local View Development"
date: 2021-10-15T12:27:17-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Although eventually, we will merge the config server UI to the light-portal view, we still have a view folder in the light-config-server repo so that UI developers can use it to try the UI and add UI components. This tutorial will show you how to start config server API to support the UI development. 

### Docker-compose

We need to start the docker-compose-integration-test.yml from the light-docker repo. This will start the MongoDB as the config server depends on it. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-integration-test.yml up
```


### Config Server

Start the config server in IDE or with fat jar with the default configuration.

```
cd ~/networknt/light-config-server
mvn clean install -Prelease
java -jar target/server.jar

```

### Start View

Start the view application

```
cd ~/networknt/light-config-server
cd view
yarn install
yarn start

```

A browser tab will open with URL point to http://localhost:3000/

### Debug and Test

Now you can debug the SPA and test the UI app. 


### Video Tutorial


{{< youtube 8BlS7K1_VKk >}}

