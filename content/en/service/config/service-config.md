---
title: "Service Config"
date: 2021-06-15T11:49:31-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
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
export CONFIG_SERVER_AUTHORIZATION=eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4c
CI6MTk0MTEyMjc3MiwianRpIjoiTkc4NWdVOFR0SEZuSThkS2JsQnBTUSIsImlhdCI6MTYyNTc2Mjc3MiwibmJmIjoxNjI1NzYyNjUyLCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny
00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOiJwb3J0YWwuciBwb3J0YWwudyIsInNlcnZpY2UiOiIwMTAwIn0.Q6BN5CGZL2fBWJk4PIlfSNXpnVyFhK6H8X4caKqxE1XAbX5UieCdXazCuwZ15wxyQJg
WCsv4efoiwO12apGVEPxIc7gpvctPrRIDo59dmTjfWH0p3ja0Zp8tYLD-5Sh65WUtJtkvPQk0uG96JJ64Da28lU4lGFZaCvkaS-Et9Wn0BxrlCE5_ta66Qc9t4iUMeAsAHIZJffOBsREFhOpC0dKSXBAyt9yuLDuD
t9j7HURXBHyxSBrv8Nj_JIXvKhAxquffwjZF7IBqb3QRr-sJV0auy-aBQ1v8dYuEyIawmIP5108LH8QdH-K8NkI1wMnNOz_wWDgixOcQqERmoQ_Q3g
export LIGHT_ENV=dev
export LIGHT_4J_CONFIG_DIR=/config
export LIGHT_CONFIG_SERVER_URI=https://localhost:8443
```

When using Kubernetes, all variables should be passed through as environment variables in the deployment. However, if you are starting the server locally, you don't want to put all of the variables to .profile as they are not easily changeable and might impact your other light-4j services developed on the same desktop. 

For me, I am putting the following to the .profile

```
export CONFIG_SERVER_CLIENT_TRUSTSTORE_LOCATION=/home/steve/networknt/light-4j/server/src/main/resources/config/bootstrap.truststore
export CONFIG_SERVER_CLIENT_TRUSTSTORE_PASSWORD=password
export CONFIG_SERVER_CLIENT_VERIFY_HOST_NAME=false
export CONFIG_SERVER_AUTHORIZATION=Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4c
CI6MTk0MTEyMjc3MiwianRpIjoiTkc4NWdVOFR0SEZuSThkS2JsQnBTUSIsImlhdCI6MTYyNTc2Mjc3MiwibmJmIjoxNjI1NzYyNjUyLCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny
00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOiJwb3J0YWwuciBwb3J0YWwudyIsInNlcnZpY2UiOiIwMTAwIn0.Q6BN5CGZL2fBWJk4PIlfSNXpnVyFhK6H8X4caKqxE1XAbX5UieCdXazCuwZ15wxyQJg
WCsv4efoiwO12apGVEPxIc7gpvctPrRIDo59dmTjfWH0p3ja0Zp8tYLD-5Sh65WUtJtkvPQk0uG96JJ64Da28lU4lGFZaCvkaS-Et9Wn0BxrlCE5_ta66Qc9t4iUMeAsAHIZJffOBsREFhOpC0dKSXBAyt9yuLDuD
t9j7HURXBHyxSBrv8Nj_JIXvKhAxquffwjZF7IBqb3QRr-sJV0auy-aBQ1v8dYuEyIawmIP5108LH8QdH-K8NkI1wMnNOz_wWDgixOcQqERmoQ_Q3g
export LIGHT_ENV=dev

```

And the following two variable might be changed from project to project. We are going to put it into the command line with -D Java Options. 

```
-Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-example-4j/rest/petstore-config-server -Dlight-config-server-uri=https://localhost:8443
```

With -Dlight-4j-config-dir we can get the externalized config folder for different service. 
With -Dlight-config-server-uri we can turn on or off config server bootstrap for the service. 

For more information on how to test config server and petstore service, you can visit the [petstore config server tutorial](/tutorial/config-server/mongo-debug/).
