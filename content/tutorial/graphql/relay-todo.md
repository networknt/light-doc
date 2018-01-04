---
title: "Relay Todo Tutorial"
date: 2018-01-03T10:31:47-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In this tutorial, we are going to port the [Relay TodoMVC example][] where the GraphQL server is replaced with
[light-graphql-4j][].  

Let's build this from [mutation example][] and make some modification. 

### Backup and Copy

First let's backup the existing relaytodo in light-example-4j/graphql folder and rebuild using the same folder
name. We can always compare the .bak folder to see if we have missed something. 

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
cd light-example-4j/graphql
mv relaytodo relaytodo.bak
cp -r mutation relaytodo
```

### pom.xml

Only artifact and name are changed.

```xml
    <artifactId>example-graphql-relaytodo</artifactId>
    <name>example-graphql-relaytodo</name>

```

### TodoSchema

There are several files to support Todo schema and they are located at 

https://github.com/networknt/light-example-4j/tree/master/graphql/relaytodo/src/main/java/com/networknt/schema

After you copy these files, you need to remove the starwars/schema folder. 


### service.yml

```yaml
- com.networknt.graphql.router.SchemaProvider:
  - com.networknt.schema.TodoSchema
```


### Relay React App

The client app is located at 

https://github.com/networknt/light-example-4j/tree/master/graphql/relaytodo/app

And this folder needs to be copied over.

### Start servers and test

Starting the frontend:

```bash
cd app
npm install
npm start
```

Starting the backend: (running on port 8080)

```bash
mvn clean install exec:exec
```

The app is now available at http://localhost:3000

Note: the Javascript app cannot auto refresh so you have to manually refresh the app to see the updates.


[light-graphql-4j]: /style/light-graphql-4j/
[Relay TodoMVC example]: https://github.com/graphql-java/todomvc-relay-java
[mutation example]: /tutorial/graphql/mutation/

