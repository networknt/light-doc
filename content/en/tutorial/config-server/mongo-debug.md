---
title: "Mongo Provider in Debug Mode"
date: 2021-05-31T00:01:01-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this document, we are going to walk through the Mongo Provider with light-config-server in debug mode. It is a great way to learn how the config server works with different providers and how the light-4j service can communicate with the config server during the startup. The generated [petstore](https://github.com/networknt/light-example-4j/tree/release/rest/petstore-maven-single) in the light-example-4j/rest will be used for the demo. 

### Preparation

Create a startup.yml file in the light-config-test to enable the config loader for the petstore service to bootstrap with the light-config-server. The config folder `petstore-config-server` can be found at this [location](https://github.com/networknt/light-config-test/tree/master/light-example-4j/rest) on Github. 


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

You need to checkout this repo to your `~/networknt` folder for the petstore application to start from the light-example-4j repo. 

```
cd ~/networknt
git clone git@github.com:networknt/light-config-test.git
git clone git@github.com:networknt/light-example-4j.git
```

Start the Mongo DB and Mongo Express with the docker-compose in the light-config-server root folder.

```
cd ~/networknt
git clone git@github.com:networknt/light-config-server.git
cd ~/networknt/light-config-server
docker-compose up -d
```

You can verify the Mongo Express with the following URL on your browser. 

```
http://localhost:8081/
```

### Start Config Server

To start the config server on the local with Mongo as the provider. Please note, the default provider is Mongo DB in the embedded config. 


```
cd ~/networknt/light-config-server
mvn clean install -Prelease
java -jar target/light-config-server-2.0.28-SNAPSHOT.jar
```

Depending on the current version in the master branch, you need to update the jar file name in the command line. 

To debug the light-config-server, you can also start the config server with your IDE in debug mode.


### Add Config File

Before we start the pet store API, we need to add a config file server.yml to the Mongo DB through Mongo Express. 

```
{
    _id: 'files/example/globals/0.0.1/dev',
    configs: [
        {
            name: 'server.yml',
            content: '\n# Server configuration\n---\n# This is the default binding address if the service is dockerized.\nip: 0.0.0.0\n\n# Http port if enableHttp is true. It will be ignored if dynamicPort is true.\nhttpPort: 49587\n\n# Enable HTTP should be false by default. It should be only used for testing with clients or tools\n# that don\'t support https or very hard to import the certificate. Otherwise, https should be used.\n# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is\n# only one protocol will be used for the server at anytime. If both http and https are true, only\n# https listener will be created and the server will bind to https port only.\nenableHttp: true\n\n# Https port if enableHttps is true. It will be ignored if dynamicPort is true.\nhttpsPort: 49588\n\n# Enable HTTPS should be true on official environment and most dev environments.\nenableHttps: false\n\n# Http/2 is enabled by default for better performance and it works with the client module\nenableHttp2: true\n\n# Keystore file name in config folder. KeystorePass is in secret.yml to access it.\nkeystoreName: server.keystore\n\n# Keystore password\nkeystorePass: ${server.keystorePass:password}\n\n# Private key password\nkeyPass: ${server.keyPass:password}\n\n# Flag that indicate if two way TLS is enabled. Not recommended in docker container.\nenableTwoWayTls: false\n\n# Truststore file name in config folder. TruststorePass is in secret.yml to access it.\ntruststoreName: server.truststore\n\n# Truststore password\ntruststorePass: ${server.truststorePass:password}\n\n# Unique service identifier. Used in service registration and discovery etc.\nserviceId: com.networknt.config-server-1.0.0\n\n# Flag to enable self service registration. This should be turned on on official test and production. And\n# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.\nenableRegistry: false\n\n# Dynamic port is used in situation that multiple services will be deployed on the same host and normally\n# you will have enableRegistry set to true so that other services can find the dynamic port service. When\n# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true\ndynamicPort: false\n\n# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup\n# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be\n# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods\nminPort: 2400\n\n# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or\n# reduced to ease firewall rules.\nmaxPort: 2500\n\n# environment tag that will be registered on consul to support multiple instances per env for testing.\n# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md\n# This tag should only be set for testing env, not production. The production certification process will enforce it.\n# environment: test1\n\n# Build Number\n# Allows teams to audit the value and set it according to their release management processes\nbuildNumber: 1.0.0'
        }
    ]
}
```

You can also add this config entry from the config server test case if you build the config server with `mvn clean install -Prelease` or explicitly run the test cases within the IDE. 


### Start Petstore Server

Before start the petstore service on the same desktop, we need to setup the following environment variables. These variables will be used to bootstrap the service, connect to the config server, load config files, cache loaded files, and start the service with the config files.

Here is a section in my .profile

```
export CONFIG_SERVER_CLIENT_TRUSTSTORE_LOCATION=/home/steve/networknt/light-4j/s
erver/src/main/resources/config/bootstrap.truststore
export CONFIG_SERVER_CLIENT_TRUSTSTORE_PASSWORD=password
export CONFIG_SERVER_CLIENT_VERIFY_HOST_NAME=false
export CONFIG_SERVER_AUTHORIZATION=eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4c
CI6MTk0MTEyMjc3MiwianRpIjoiTkc4NWdVOFR0SEZuSThkS2JsQnBTUSIsImlhdCI6MTYyNTc2Mjc3MiwibmJmIjoxNjI1NzYyNjUyLCJ2ZXJzaW9uIjoiMS4wIiwiY2xpZW50X2lkIjoiZjdkNDIzNDgtYzY0Ny
00ZWZiLWE1MmQtNGM1Nzg3NDIxZTcyIiwic2NvcGUiOiJwb3J0YWwuciBwb3J0YWwudyIsInNlcnZpY2UiOiIwMTAwIn0.Q6BN5CGZL2fBWJk4PIlfSNXpnVyFhK6H8X4caKqxE1XAbX5UieCdXazCuwZ15wxyQJg
WCsv4efoiwO12apGVEPxIc7gpvctPrRIDo59dmTjfWH0p3ja0Zp8tYLD-5Sh65WUtJtkvPQk0uG96JJ64Da28lU4lGFZaCvkaS-Et9Wn0BxrlCE5_ta66Qc9t4iUMeAsAHIZJffOBsREFhOpC0dKSXBAyt9yuLDuD
t9j7HURXBHyxSBrv8Nj_JIXvKhAxquffwjZF7IBqb3QRr-sJV0auy-aBQ1v8dYuEyIawmIP5108LH8QdH-K8NkI1wMnNOz_wWDgixOcQqERmoQ_Q3g
export LIGHT_ENV=dev
```

You can also pass in all the variables through the -D option on the Java command line, but some duplicated variables need to be set up for every service. I normally only put two variables to the -D option per project. One is to specify the externalized config folder, and the other is to turn on the config server bootstrap instead of the file system config loader. 


```
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-example-4j/rest/petstore-config-server -Dlight-config-server-uri=https://localhost:8443 -jar target/server.jar 
```


