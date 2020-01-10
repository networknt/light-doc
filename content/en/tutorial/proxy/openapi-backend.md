---
title: "Openapi Backend"
date: 2017-12-16T10:43:39-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to create a backend service and start three instances to demo the [light-proxy][] load balance between them. 

The backend service will have two endpoints: one get and one post to demo different scenarios in term of proxy functionality. This server is built on top of light-rest-4j with OpenAPI 3.0 specification.

There is another [backend service][] that is built on top of Swagger 2.0 specification for users who are still working on Swagger 2.0 specification for some reason. 

### Backend service specification and light-codegen config.json

You can find the OpenAPI 3.0 specification and config.json in [model-config][]

Regarding how to create OpenAPI 3.0 Specification, please refer to [swagger editor][]

### Generate the backend service

Given the OpenAPI specification and config.json, we are going to use light-codegen to generate the project and update the service with a meaningful response based on the specification.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-config-test.git
cd light-codegen
mvn clean install -DskipTests
cd ~/networknt
rm -rf light-example-4j/rest/openapi/proxy-backend
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/proxy-backend -m model-config/rest/openapi/proxy-backend/openapi.yaml -c model-config/rest/openapi/proxy-backend/config.json
```

The newly generated project can be found in [light-example-4j][], and you can compare it with your own generated code.

### Update handlers

Now with the project generated, let's update the two handlers to output some meaningful info when the endpoints are called. 

Here is the GetDataGetHandler.java

```java
package com.networknt.backend.handler;

import com.networknt.config.Config;
import com.networknt.server.Server;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;

import java.util.HashMap;
import java.util.Map;

public class GetDataGetHandler implements HttpHandler {
    static final boolean enableHttp2 = Server.config.isEnableHttp2();
    static final boolean enableHttps = Server.config.isEnableHttps();
    static final int httpPort = Server.config.getHttpPort();
    static final int httpsPort = Server.config.getHttpsPort();

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        Map<String, Object> examples = new HashMap<>();
        examples.put("key", "key1");
        examples.put("value", "value1");
        examples.put("enableHttp2", enableHttp2);
        examples.put("enableHttps", enableHttps);
        examples.put("httpPort", httpPort);
        examples.put("httpsPort", httpsPort);
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(examples));
    }
}
```
Here is the PostDataPostHandler.java

```java

package com.networknt.backend.handler;

import com.networknt.body.BodyHandler;
import com.networknt.config.Config;
import com.networknt.server.Server;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.HttpString;
import org.apache.commons.lang3.StringEscapeUtils;

import java.util.HashMap;
import java.util.Map;

public class PostDataPostHandler implements HttpHandler {
    static final boolean enableHttp2 = Server.config.isEnableHttp2();
    static final boolean enableHttps = Server.config.isEnableHttps();
    static final int httpPort = Server.config.getHttpPort();
    static final int httpsPort = Server.config.getHttpsPort();

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        Map<String, Object> body = (Map<String, Object>)exchange.getAttachment(BodyHandler.REQUEST_BODY);
        Map<String, Object> examples = new HashMap<>(body);
        examples.put("enableHttp2", enableHttp2);
        examples.put("enableHttps", enableHttps);
        examples.put("httpPort", httpPort);
        examples.put("httpsPort", httpsPort);
        exchange.getResponseHeaders().add(new HttpString("Content-Type"), "application/json");
        exchange.getResponseSender().send(Config.getInstance().getMapper().writeValueAsString(examples));
    }
}
```
As you can see, the output contains information on which port the response is coming from, as we will start multiple instances of the backend for the proxy.


### Start backend services

If you have multiple computers, you can start multiple instances that listening on the same port on each computer. Otherwise, start multiple instances that listening to different ports on one computer. Here we are going to start three instances on the same computer and let them listen to 8081, 8082 and 8083 on https. Let's disable the HTTP for now as it is never used on production.

If you generate the proxy-backend service, you should have the project on your local light-example-4j/rest/openapi folder already. If not, let's clone the repo and compile it locally.

```
cd ~/networknt
git clone https://github.com/networknt/light-example-4j.git
```
In order to start three instances with different https ports, we need to overwrite the `httpsPort` in server.yml, which is located at proxy-backend/src/main/resources/config folder with an external values.yml file. The following is the default one generated based on the config.json file. As you can see, HTTP is disabled, and https is enabled and listen to 8443 port. 

```yaml


# Server configuration
---
# This is the default binding address if the service is dockerized.
ip: 0.0.0.0

# Http port if enableHttp is true.
httpPort:  8080

# Enable HTTP should be false on official environment.
enableHttp: false

# Https port if enableHttps is true.
httpsPort:  8443

# Enable HTTPS should be true on official environment.
enableHttps: true

# Http/2 is enabled by default.
enableHttp2: true

# Keystore file name in config folder. KeystorePass is in secret.yml to access it.
keystoreName: tls/server.keystore

# Flag that indicate if two way TLS is enabled. Not recommended in docker container.
enableTwoWayTls: false

# Truststore file name in config folder. TruststorePass is in secret.yml to access it.
truststoreName: tls/server.truststore

# Unique service identifier. Used in service registration and discovery etc.
serviceId: com.networknt.backend-1.0.0

# Flag to enable service registration. Only be true if running as standalone Java jar.
enableRegistry: false

# environment tag that will be registered on consul to support multiple instances per env for testing.
# https://github.com/networknt/light-doc/blob/master/docs/content/design/env-segregation.md
# This tag should only be set for testing env, not production. The production certification process will enforce it.
# environment: test1

```

The externalized values.yml files for httpsPort 8081, 8082 and 8083 can be found at https://github.com/networknt/light-config-test/tree/master/light-example-4j/rest/openapi/proxy-backend and the repository has been checked out locally. 


Now let's start the first instance in a new terminal.

```
cd ~/networknt/light-example-4j/rest/openapi/proxy-backend
mvn clean install -Prelease
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-example-4j/rest/openapi/proxy-backend/8081 -jar target/backend-1.0.0.jar
```

Please note that the configuration folder might be different on your computer. 

Now from another terminal, you can issue a curl command to ensure the server is running and listening on 8081 on https/http2.


```
curl -k https://localhost:8081/v1/getData
```

Result:

```json
{"enableHttp2":true,"httpPort":8080,"enableHttps":true,"value":"value1","httpsPort":8081,"key":"key1"}
```

You can also test the post request with the following command line.

```
curl -k -X POST https://localhost:8081/v1/postData   -H 'content-type: application/json'   -d '{"key":"key1", "value": "value1"}'
```

And the request should be something like this.

```json
{"enableHttps":true,"value":"value1","httpsPort":8081,"key":"key1","enableHttp2":true,"httpPort":8080}
```

Now let's start another service in another terminal with 8082 config folder to overwrite the httpsPort to 8082.

```
cd ~/networknt/light-example-4j/rest/openapi/proxy-backend
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-example-4j/rest/openapi/proxy-backend/8082 -jar target/backend-1.0.0.jar
```

Please note that the configuration folder might be different on your computer. 

Test the second instance with command:

```
curl -k https://localhost:8082/v1/getData
```

Now let's start the third instance with port number 8083.

```
cd ~/networknt/light-example-4j/rest/openapi/proxy-backend
java -Dlight-4j-config-dir=/home/steve/networknt/light-config-test/light-example-4j/rest/openapi/proxy-backend/8083 -jar target/backend-1.0.0.jar
```

Please note that the configuration folder might be different on your computer. 

Test the third instance with command:

```
curl -k https://localhost:8083/v1/getData
```

At this moment, we have three backend service instances running, and they are listening to 8081, 8082, and 8083 on the https/http2 protocol.

[light-proxy]: https://github.com/networknt/light-proxy
[backend service]: /tutorial/proxy/swagger-backend/
[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi/proxy-backend
[swagger editor]: /tool/swagger-editor/
[light-example-4j]: https://github.com/networknt/light-example-4j/tree/master/rest/openapi/proxy-backend
