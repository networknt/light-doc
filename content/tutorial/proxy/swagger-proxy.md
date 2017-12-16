---
title: "Swagger Proxy"
date: 2017-12-16T10:44:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


In this second par of the tutorial, we are going to use Swagger 2.0 RESTful 
API as an example to show how to deployed the light-proxy and enable all sorts 
of middleware handlers through configurations. We also explore standalone deployment 
and Docker deployment. 

With specification handler, security handler and validation handler change, you
can easily update the configuration to make the same instance of light-proxy
server work with OpenAPI 3.0 or GraphQL backend. The default configuration is for 
RESTful Swagger 2.0 or OpenAPI 3.0 backend as light-rest-4j middleware handlers are 
wired in the request/response chain.

To make it simple, we will build a backend service with light-rest-4j with a
swagger specification file defined in [model-config][] 
repo. In real production environment, the backend service can be built with other 
Java framework or other languages like Nodejs, Go etc.

If you have an existing API backend up and running already, then you can continue.
Otherwise, follow this [backend tutorial][]
to create a backend service and start multiple instances.

## light-proxy configuration

With the backend services ready, let's clone the light-proxy and make some configuration
changes. 

```
cd ~/networknt
git clone https://github.com/networknt/light-proxy.git
```

By default, the light-proxy doesn't have any config files packaged into the src/main/resources/config
folder as production deployment will have all the config files externalized. Now, let's
copy the config files from src/test/resources/config and make some modifications.

```
cd ~/networknt/light-proxy/src/main/resources
mkdir config
cp -r ../../test/resources/config/* config 
```

Now you can add a proxy.yml in light-proxy/src/main/resources/config folder as following.

```yaml
# Reverse Proxy Handler Configuration

# If HTTP 2.0 protocol will be used to connect to target servers
http2Enabled: true

# If TLS is enabled when connecting to the target servers
httpsEnabled: true

# Target URIs
hosts: https://localhost:8081,https://localhost:8082,https://localhost:8083

# Connections per thread to the target servers
connectionsPerThread: 20

# Max request time in milliseconds before timeout
maxRequestTime: 10000
``` 

As we are going to enable Security and Validator based on swagger.json from backend server,
let's copy the swagger.json from light-example-4j/rest/swagger/proxy-backend/src/main/resources/config folder.

```
cd ~/networknt
cp light-example-4j/rest/swagger/proxy-backend/src/main/resources/config/swagger.json light-proxy/src/main/resources/config/
```

## start light-proxy and test

```
cd ~/networknt/light-proxy
mvn clean install -DskipTests
mvn exec:exec
```

Given the light-proxy server.yml only have https enabled and listening to port 8080, let's
use curl to test it.

```
curl -k https://localhost:8080/v1/getData
```

And we get result:

```
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8082,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8083,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8081,"key":"key1"}
curl -k https://localhost:8080/v1/getData
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8082,"key":"key1"}
```

As you can see the proxy server is working on https connection to the backend services. And 
the response is from different instances for each request due to load balance. You can stop 
one of the three instances and the proxy will automatically failover to working instances.

Also, you can test the post request with the following command line.

```
curl -k -X POST \
  https://localhost:8080/v1/postData \
  -H 'content-type: application/json' \
  -d '{"key":"key1", "value": "value1"}'
```

Result: 

```json
{"enableHttps":true,"value":"value1","httpsPort":8083,"key":"key1","enableHttp2":true,"httpPort":8080}
```

Above steps shows how to start the backend services and to config the light-proxy so that it
can pass through the requests to the backend instances in a round robin fashion. 

For light-proxy configuration, we just updated it in the app config folder to add a proxy.yml 
and copy over the swagger.json from proxy-backend service. This will enable the proxy 
server to do security verification (disabled by default) based on JWT token and scopes. 

The reason we do in app configuration is to show you how this has been done within the application
context. In the normal way, you should have light-proxy jar file or Docker image as deliver
package and all configuration should be done externally. 

For information on how to pass the configuration files into a standalone service please refer
to this [config module][]

For information on how to pass the configuration files into a Docker container please refer
to this [config module][]

For information on how to enable features of light-proxy with config files, please refer to
light-proxy [configuration][] 

[model-config]: https://github.com/networknt/model-config/tree/master/rest/swagger/proxy-backend

[backend tutorial]: /tutorial/proxy/swagger-backend/
[config module]: /concern/config/
[configuration]: /service/proxy/configuration/
