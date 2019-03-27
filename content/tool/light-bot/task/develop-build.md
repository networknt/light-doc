---
title: "Develop Build"
date: 2019-03-27T13:26:36-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is a task that is used to build all related repositories from a specific branch and perform unit, integration as well as end-to-end tests. It should be set up as a scheduled process on build server for now, and in the future, it should be called by agent server once the controller receives push event from github.com 

This task is mostly used by the developer to build all dependencies on their local from master branch daily. In this case, they will skip the test so that it runs faster and doesn't need to start dependent docker containers like Kafka, MySQL, etc.

We also run this task on our build server several times per day with all tests enabled so that we can detect broken PRs immediately. For most developers, they will test the repository they are working on to ensure all tests are passed. However, there are situations that one change in light-4j might break light-eventuate-4j. 

Before running the following build for networknt repositories with test enabled, you need to start the following dependent docker-compose files as integration tests need these dependencies. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-integration-test.yml up -d
# after one minute, start the tram cdcserver as it needs Kafka to be running
docker-compose -f docker-compose-cdcserver-for-tram.yml up -d
```

Here are the steps in the task.

* it clones all projects defined in the checkout section into a workspace.
* if the workspace exists, then it will iterate each repository to switch to the branch specified in the checkout and pull from the remote. If none of the repo has been updated, then skip the rest of the process.
* build each repository with "maven clean install". This command runs unit tests and integration tests for each repo.
* start one or more microservices and run end-to-end test cases.


You can find the configuration examples in [light-config-test][] from checkout and develop-build folders. 


Here is an example of develop-build.yml used for networknt repositories and examples.

```yaml
workspace: developbuild_1_6_x
# indicate if you want to skip checkout. yes if you know that all repositories are just checkout
# or the last time the build was failed and you just want to retry without changing anything.
skip_checkout: false
# Just checkout the repositories for backup or some other tasks
skip_build: false
# If this value is set to true, then only checkout and build will be called. It is
# very useful if you just want to install the develop branch modules into your .m2
# local repo. Also, some of our customers have constraint test environment to run
# unit and integration test only and another environment to run all of tests.
skip_test: false
# skip copy file if necessary.
skip_copyFile: true
# skip copyWildcardFile
skip_copyWildcardFile: true
# skip start
skip_start: true

# Execute this set of tasks in order
# setup is called before any of the tasks
# teardown and stop are called regardless of a succesful/failed light-bot run
tasks:
  # one set of operations
  checkoutSetOne: checkout
  buildSetOne: build
  testProjects: test     
  copyProjectFiles: copyFile 
  copyWildcardFile: copyWildcardFile

checkout:
  checkoutSetOne: 
    - branch: master
      skip: false
      repository:
      - git@github.com:networknt/light-4j.git
      - git@github.com:networknt/json-overlay.git
      - git@github.com:networknt/openapi-parser.git
      - git@github.com:networknt/light-rest-4j.git
      - git@github.com:networknt/light-graphql-4j.git
      - git@github.com:networknt/light-hybrid-4j.git
      - git@github.com:networknt/light-codegen.git
      - git@github.com:networknt/light-eventuate-4j.git
      - git@github.com:networknt/light-tram-kafka.git
      - git@github.com:networknt/light-tram-4j.git
      - git@github.com:networknt/light-saga-4j.git
      - git@github.com:networknt/light-session-4j.git
      - git@github.com:networknt/light-spa-4j.git
      - git@github.com:networknt/light-proxy.git
      - git@github.com:networknt/light-router.git
      - git@github.com:networknt/light-oauth2.git
      - git@github.com:networknt/light-tokenization.git
      - git@github.com:networknt/light-portal.git
      - git@github.com:networknt/light-example-4j.git
      - git@github.com:networknt/light-docker.git
      - git@github.com:networknt/light-consumer-4j.git
      - git@github.com:networknt/http2client-benchmark.git
      - git@github.com:networknt/microservices-framework-benchmark.git
      - git@github.com:networknt/light-config-server.git
      - git@github.com:networknt/light-spring-boot.git

build:
  buildSetOne:
    skip: false
    project:
    - light-4j
    - json-overlay
    - openapi-parser
    - light-rest-4j
    - light-graphql-4j
    - light-hybrid-4j
    - light-codegen
    - light-eventuate-4j
    - light-tram-kafka
    - light-tram-4j
    - light-saga-4j
    - light-session-4j
    - light-spa-4j
    - light-proxy
    - light-router
    - light-oauth2
    - light-tokenization
    - light-portal
    - light-portal/api-certification
    - light-portal/host-menu
    - light-portal/schema-form
    - light-portal/user-management
    - light-portal/hybrid-command
    - light-portal/hybrid-query
    - light-consumer-4j
    - http2client-benchmark/httpserver
    - http2client-benchmark/lightclient
    - microservices-framework-benchmark/light-4j  
    - light-config-server
    - light-spring-boot
    - light-example-4j/rest/openapi/petstore
    - light-example-4j/rest/openapi/proxy-backend
    - light-example-4j/rest/openapi/cors
    - light-example-4j/rest/openapi/ms-aggregate/aa/generated
    - light-example-4j/rest/openapi/ms-aggregate/aa/https
    - light-example-4j/rest/openapi/ms-aggregate/ab/generated
    - light-example-4j/rest/openapi/ms-aggregate/ab/https
    - light-example-4j/rest/openapi/ms-aggregate/ac/generated
    - light-example-4j/rest/openapi/ms-aggregate/ac/https
    - light-example-4j/rest/openapi/ms-aggregate/ad/generated
    - light-example-4j/rest/openapi/ms-aggregate/ad/https
    - light-example-4j/rest/swagger/petstore
    - light-example-4j/rest/swagger/proxy-backend
    - light-example-4j/rest/swagger/ms_chain/api_a/httpschain
    - light-example-4j/rest/swagger/ms_chain/api_b/httpschain
    - light-example-4j/rest/swagger/ms_chain/api_c/httpschain
    - light-example-4j/rest/swagger/ms_chain/api_d/httpschain
    - light-example-4j/rest/swagger/database/generated
    - light-example-4j/rest/swagger/database/connection
    - light-example-4j/rest/swagger/database/oracle
    - light-example-4j/rest/swagger/database/postgres
    - light-example-4j/rest/swagger/database/queries
    - light-example-4j/rest/swagger/database/query
    - light-example-4j/rest/swagger/database/test
    - light-example-4j/rest/swagger/database/updates
    - light-example-4j/client/standalone
    - light-example-4j/client/tomcat
    - light-example-4j/client/consul
    - light-example-4j/client/tableau
    - light-example-4j/graphql/helloworld
    - light-example-4j/graphql/starwars
    - light-example-4j/graphql/mutation
    - light-example-4j/graphql/mutation-idl
    - light-example-4j/graphql/relaytodo
    - light-example-4j/graphql/subscription
    # light-eventuate-4j
    - light-example-4j/eventuate/account-management
    - light-example-4j/eventuate/todo-list
    # light-tram-4j
    # - light-example-4j/tram/light-tram-todolist/single-module
    - light-example-4j/tram/light-tram-todolist/multi-module
    # light-saga-4j
    # - light-example-4j/saga/light-saga-cutomers-and-orders
    # middleware-performance
    - light-example-4j/middleware-performance/service-config
    - light-example-4j/middleware-performance/endpoint-individual
    - light-example-4j/middleware-performance/endpoint-source
# This section defines end-to-end tests with real live servers, if you want to skip
# these tests, please change the same level skipE2ETest to true.
test:
  rest-openapi-petstore:
    server:
    - path: light-example-4j/rest/openapi/petstore
      cmd: target/petstore-3.0.1.jar
    request:
    - host: https://localhost:8443
      path: "/v1/pets/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": Jessica Right
          "$.id": 1
  rest-openapi-proxy-backend:
    server:
    - path: light-example-4j/rest/openapi/proxy-backend
      cmd: target/backend-1.0.0.jar
    request:
    - host: https://localhost:8081
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8081
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
  rest-openapi-cors:
    server:
    - path: light-example-4j/rest/openapi/cors
      cmd: target/cors-1.0.1.jar
    request:
    - host: https://localhost:8443
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.[0].key": key1
          "$.[1].key": key2
          "$.[0].value": value1
          "$.[1].value": value2
    - host: https://localhost:8443
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8443
      path: "/v1/postData"
      method: options
      header:
        "Origin": http://example.com
        "Access-Control-Request-Method": POST
        "Access-Control-Request-Headers": X-Requested-With
      response:
        status: 200
  rest-openapi-aggregate:
    server:
    - path: light-example-4j/rest/openapi/ms-aggregate/ad/https
      cmd: target/ad-1.0.0.jar
    - path: light-example-4j/rest/openapi/ms-aggregate/ac/https
      cmd: target/ac-1.0.0.jar
    - path: light-example-4j/rest/openapi/ms-aggregate/ab/https
      cmd: target/ab-1.0.0.jar
    - path: light-example-4j/rest/openapi/ms-aggregate/aa/https
      cmd: target/aa-1.0.0.jar
    request:
    - host: https://localhost:7441
      path: "/v1/data"
      method: get
      response:
        status: 200
        body:
          "$.length()": 8
  rest-swagger-petstore:
    server:
    - path: light-example-4j/rest/swagger/petstore
      cmd: target/petstore-2.0.0.jar
    request:
    - host: https://localhost:8443
      path: "/v2/pet/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": doggie
          "$.id": 123456789
  rest-swagger-proxy-backend:
    server:
    - path: light-example-4j/rest/swagger/proxy-backend
      cmd: target/backend-1.0.0.jar
    request:
    - host: https://localhost:8081
      path: "/v1/getData"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
    - host: https://localhost:8081
      path: "/v1/postData"
      method: post
      header:
        "content-type": application/json
      body: "{\"key\":\"key1\",\"value\":\"value1\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.enableHttp2": true
          "$.enableHttps": true
          "$.httpsPort": 8081
          "$.key": key1
          "$.value": value1
  rest-swagger-chain:
    server:
    - path: light-example-4j/rest/swagger/ms_chain/api_d/httpschain
      cmd: target/apid-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_c/httpschain
      cmd: target/apic-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_b/httpschain
      cmd: target/apib-1.0.0.jar
    - path: light-example-4j/rest/swagger/ms_chain/api_a/httpschain
      cmd: target/apia-1.0.0.jar
    request:
    - host: https://localhost:7441
      path: "/v1/data"
      method: get
      response:
        status: 200
        body:
          "$.length()": 8
  graphql-helloworld:
    server:
    - path: light-example-4j/graphql/helloworld
      cmd: target/starwars-1.0.1.jar
    request:
    - host: http://localhost:8080
      path: "/graphql"
      method: post
      header:
        "content-type": application/json
      body: "{\"query\":\"{ hello }\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.data.hello": world
  graphql-mutation:
    server:
    - path: light-example-4j/graphql/mutation
      cmd: target/starwars-1.0.1.jar
    request:
    # query
    - host: http://localhost:8081
      path: "/graphql"
      method: post
      header:
        "content-type": application/json
      body: "{\"query\":\"{ numberHolder { theNumber }}\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.data.numberHolder.theNumber": 6
    # mutation
    - host: http://localhost:8081
      path: "/graphql"
      method: post
      header:
        "content-type": application/json
      body: "{\"query\":\"mutation { changeTheNumber(newNumber: 4) { theNumber }}\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.data.changeTheNumber.theNumber": 4
    # query again
    - host: http://localhost:8081
      path: "/graphql"
      method: post
      header:
        "content-type": application/json
      body: "{\"query\":\"{ numberHolder { theNumber }}\"}"
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.data.numberHolder.theNumber": 4
    # subscription test has two browser windows and cannot be tested here.

  # dynamic port testing with two instances  
  dynamic-port-petstore:
    server:
    - path: light-example-4j/rest/openapi/dyanmic-port
      cmd: target/petstore-3.0.1.jar
    - path: light-example-4j/rest/openapi/dyanmic-port
      cmd: target/petstore-3.0.1.jar
    request:
    - host: https://localhost:2400
      path: "/v1/pets/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": Jessica Right
          "$.id": 1
    - host: https://localhost:2401
      path: "/v1/pets/111"
      method: get
      response:
        status: 200
        header:
          content-type: application/json
        body:
          "$.name": Jessica Right
          "$.id": 1
          
# copy individual files
copyFile:

# copy wildcard files
copyWildcardFile:

```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot