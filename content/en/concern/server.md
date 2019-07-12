---
title: "Server"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

This module is responsible for managing the life cycle of the embedded Undertow core HTTP server. It starts the server and initializes all middleware handlers/plugins along with a route handler provider when the handler chain is defined in the `service.yml`  or OrchestrationHandler when the handler chains are defined in the `handler.yml`. It gracefully stops the server and allows the resources to be released and all in-flight requests to be processed before the server instance is deregistered on the service registry even someone clicks CTRL+C on the terminal.

### Startup Hooks

During server startup, a list of startup hooks is called to initialize the context and environment for the server instance. For example, creating a database connection pool, initialize cache, etc.

All startup hooks must implement the following interface.

```
public interface StartupHookProvider {
    void onStartup();
}
```

Startup hooks are loaded during the server startup from [service][] module and the config file that binds all startupHooksProviders is the `service.yml` file. This file is in src/main/resources/config folder in your generated project from [light-codegen][], and it should be externalized for any official deployment environment. 

Here is an example.

```
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.StartupHookProvider:
  - com.networknt.server.Test1StartupHook
  - com.networknt.server.Test2StartupHook
```

### Shutdown Hooks

During server shutdown, a list of shutdown hooks is called to clean up and release resources for the server instance. For example, close all the database connections in the connection pool and release any resource allocation.

All shutdown hooks must implement the following interface.

```
public interface ShutdownHookProvider {
    void onShutdown();
}
```

Shutdown hooks are loaded during the server startup from the service module, and the configuration is defined in the `service.yml` file as startupHookProviders. Here is an example config section. These hooks will be invoked based on the sequence defined in the config file right before the server is shut down after the grace period is passed and all pending requests are handled. 
 
```
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
- com.networknt.server.ShutdownHookProvider:
  - com.networknt.server.Test1ShutdownHook

```

### Middleware Handler vs Business Handler

Light-4j has two different types of handlers: middleware handler and business handler. 

Middleware handler doesn't return anything except errors as a response but only manipulate the request or the response in the request/response chain. Each middleware handler handles the request/response and then pass the request/response to the next middleware handler in the chain. 

Business handler is the final handler in the chain that runs the business logic and return the real business response to the caller. 

For more information about handlers, please visit the [handler][] cross-cutting concern. 


### Route Provider

[light-codegen][] generates all the business domain handlers and corresponding test cases based on the specifications (OpenAPI 3.0, Swagger 2.0, GraphQL IDL and Hybrid Schema). The generated handlers output examples defined in the specification and test cases are commented out. The generator also generates a HandlerProvider class to wire in business domain handlers together into the request/response chain. These business handlers are located after request middleware handlers and before response middleware handlers. This class is loaded by the server during the server startup, and the binding is configured in the `service.yml` file. 

Here is an example of the config.

```
# HandlerProvider implementation
- com.networknt.handler.HandlerProvider:
  - com.networknt.petstore.PathHandlerProvider
```


### Configuration

server.yml is the configuration file for the server module to control the server behavior. 

Here is an example of server.yml

```
# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true. It will be ignored if dynamicPort is true.
httpPort:  8080

# Enable HTTP should be false by default. It should be only used for testing with clients or tools
# that don't support https or very hard to import the certificate. Otherwise, https should be used.
# When enableHttp, you must set enableHttps to false, otherwise, this flag will be ignored. There is
# only one protocol will be used for the server at anytime. If both http and https are true, only
# https listener will be created and the server will bind to https port only.
enableHttp: false

# Https port if enableHttps is true. It will be ignored if dynamicPort is true.
httpsPort:  8443

# Enable HTTPS should be true on official environment and most dev environments.
enableHttps: true

# Http/2 is enabled by default for better performance and it works with the client module
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: tls/server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: tls/server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.petstore-3.0.1

# Flag to enable self service registration. This should be turned on on official test and production. And
# dyanmicPort should be enabled if any orchestration tool is used like Kubernetes.
enableRegistry: false

# Dynamic port is used in situation that multiple services will be deployed on the same host and normally
# you will have enableRegistry set to true so that other services can find the dynamic port service. When
# deployed to Kubernetes cluster, the Pod must be annotated as hostNetwork: true
dynamicPort: false

# Minimum port range. This define a range for the dynamic allocated ports so that it is easier to setup
# firewall rule to enable this range. Default 2400 to 2500 block has 100 port numbers and should be
# enough for most cases unless you are using a big bare metal box as Kubernetes node that can run 1000s pods
minPort: 2400

# Maximum port rang. The range can be customized to adopt your network security policy and can be increased or
# reduced to ease firewall rules.
maxPort: 2500

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

```
To obtain flexible adaptations to the user's needs and extract max performance, parts of undertow server options also can be configured in server.yml since release 1.6.x. 

Here is an example that can be appended to the above configuration file.
```
 # Buffer size of undertow server. 
 bufferSize: 10000
 
 # Number of IO thread 
 ioThreads: 1
 
 # Number of worker thread
 workerThreads: 100
 
 # Backlog size
 backlog: 10000
 
 # Flag to set UndertowOptions.ALWAYS_SET_DATE
 alwaysSetDate: false
 
 # Server string used to mark the server
 serverString: TEST 
 
 # Flag to set UndertowOptions.ALLOW_UNESCAPED_CHARACTERS_IN_URL.
 # Please note that this option widens the attack surface and attacker can potentially access your filesystem. 
 # This should only be used on an internal server and never be used on a server accessed from the Internet. allowUnescapedCharactersInUrl: true
```
The configuration of server options works follows the rules:
1. All server options will be set to default values when users do not provide server options.

2. When the server options are provided by the user but these values are invalid, the default values are used and a warning is added in the log.

    Invalid situations are as follows:

    * ioThreads <= 0
    * workerThreads <= 0
    * bufferSize <= 0
    * logback <= 0
    * serverString == null || serverString.equals("")

3. Using the values provided by the user when the server options are provided and these server options are valid

Please note that improper settings of server options can have a significant impact on overall performance. Be careful to configure these values.

### Config Server

If you want the server to load configuration files from the config server, the following environment variable must be set in the java -jar as option or pass to Docker container as environment variables.

* light-env
* light-config-server-uri

Once the server starts, it will access the [light-config-server](https://github.com/networknt/light-config-server) instance to get a map of key/value pairs that match the values.yml config files for the particular service for the environment and the framework version. The key/value map will be injected by the [config][] module automatically into the config file templates in /config folder which is specified by light-4j-config-dir system properties or src/main/resources/config folder. 

If you are deploying hundreds or thousands of services, it would be much easier to manage the configurations with light-config-server which supports hierarchical config file management with multiple GitHub organizations and repositories as well as encryption and decryption values between the config server and services inside the containers.



### Service Registry

If enableRegistry is true in server.yml, then the server will register itself to Consul or Zookeeper whichever is configured in service.yml. The self-registration is used in data center deployment which you might have several Java instances running on the same host. It is also used in Kubernetes cluster as all the services are running on the host network to take advantages of direct service discovery. 

For service registration, you can specify dynamicPort to true so that the server module will try to bind a port in a range defined between minPort and maxPort. 

* Standalone Application

If the server is started as a standalone Java application with java -jar command line, there is no extra work that needs to be done. If the dynamic port is enabled and a range of ports are specified, then the server will try to bind the first available port on the host. 

* Docker Compose

When starting the service in a docker container with a docker-compose.yml file, the service cannot find the host IP address to register itself. It knows the port number as it tries to bind one by one in a configured range. This requires us to pass the DOCKER_HOST_IP to the STATUS_HOST_IP environment variable for the server instance. In some instances, for example, Kafka consumer or streams applications, you cannot use the docker overlay network, and network_mode need to be set as host. 

Here is an example of a docker-compose.yml file. 

```
# testnet test1 docker-compose
version: "2"
services:
  chainwriter:
    image: networknt/com.networknt.chainwriter-1.0.0:latest
    volumes:
      - ./test1/chain-writer:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host
  chainreader:
    image: networknt/com.networknt.chainreader-1.0.0:latest
    volumes:
      - ./test1/chain-reader:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
  # there is only one instance of token reader and it is allocated at test1
  tokenreader:
    image: networknt/com.networknt.tokenreader-1.0.0:latest
    volumes:
      - ./test1/token-reader:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
  # there is only one instance of kyc reader and it is allocated at test1
  kycreader:
    image: networknt/com.networknt.kycreader-1.0.0:latest
    volumes:
      - ./test1/kyc-reader:/config
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
```

On a Linux server, we usually set the DOCKER_HOST_IP in the .profile file so that all containers running on the same host can get the same IP. 

* Kubernetes

When deploying to a Kubernetes cluster or Openshift cluster, the situation is very similar to Docker Compose. You need to use the host network and pass the host IP to the container through Kubernetes downward API in the deployment YAML file. 

Here is an example of deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apia-deployment
  labels:
    app: apia
spec:
  replicas: 6
  selector:
    matchLabels:
      app: apia
  template:
    metadata:
      labels:
        app: apia
    spec:
      hostNetwork: true
      containers:
      - name: apia
        image: networknt/com.networknt.apia-1.0.0
        env:
        - name: STATUS_HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
```

As you can see in the spec section that the `hostNetwork` is true and the environment variable STATUS_HOST_IP is passed in as status.hostIP from the downward API of Kubernetes. 


### Port Allocation

If you are using the service registry, you should only use the HTTPS port and disable the HTTP port all the time. In fact, although we support HTTP connection, we recommend using HTTPS all the time even in the development environment as TLS is supported out of the box from the [light-codegen][]. 

When you are trying to sign the port number manually, be sure that there is no conflict on the same host. For some services running inside another container or service platform, there is a way to pass the HTTP or HTTPS port into the light-4j service from an environment variable. One example is to deploy light-4j service into Azure App Service platform. The port number must be allocated by the Azure App Service and pass to the internal service. 

The following environment variables are supported. 

```
httpPort
httpsPort
```

By setting above environment variables, the server.yml settings are overwritten. Please note, that you only need to set one port number and only enable HTTP or HTTPS. 

For more info about setting port number with environment variables, please refer to https://github.com/networknt/light-4j/issues/210

Since release 1.5.29, we have updated the [config][] module to support config file templates and environment variables to replace values in the config file in general. We don't need the above workaround anymore. However, the feature is kept for backward compatibility. 

### Gracefully Shutdown

As we are using the service registry, discovery and client-side load balance, our client module maintains a list of live services per serviceId interested. If any service is shutting down, it takes a short while to propagate to all clients. To handle in-flight request from the clients that don't know the server instance is down, the server has to send a shutdown signal to the registry(Consul or Zookeeper) and keep processing for at least 20 to 30 seconds and then shut down. Within this period of time, all client should have known the service instance is gone and won't send any new request to it.


### TLS Hostname Verification

For testing, we can disable the hostname verification on the [client][] for the certificate; however, it is recommended that on production, hostname verification should be turned on to eliminate man-in-the-middle attacks. If possible, the two-way TLS should be enabled on both client.yml and server.yml to maximize the security. For more information on the verifyHostname setup, plese visit [client][] module.

To get the certificates, you have two options:

* Buy certificates from a CA like VeriSign.
* Setup a CA in your organization and use OpenSSL to generate certificates.

For more information

http://stackoverflow.com/questions/29546834/trust-not-trusted-certificates-and-skip-hostname-verification/29547114#29547114
https://www.owasp.org/index.php/Certificate_and_Public_Key_Pinning

[service]: /concern/service/
[light-codegen]: /tool/light-codegen/
[handler]: /concern/handler/
[config]: /concern/config/
[client]: /concern/client/
