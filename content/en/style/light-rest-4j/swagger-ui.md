---
title: "Swagger UI"
date: 2022-05-12T22:02:00-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

In light-rest-4j, there is a specification module. It contains several handlers to render Swagger UI for light-rest-4j services including http-sidecar and light-gateway to assist users to test the API from the browser. 

### Handlers

##### SpecSwaggerUIHandler

The main handler is called SpecSwaggerUIHandler and it will return an HTML file to render the Swagger UI. The HTML refers to a url for the OpenAPI specification file on the server as "/spec.yaml".

##### SpecDisplayHandler

The /spec.yaml endpoimt is served by SpecDisplayHandler and it uses the specificatiom.yml config file to locate the openapi.yaml specification file in the config folder.

Here is the default specification.yml in the module. 

```
---
# Specification type and file name definition
# The filename with path of the specification file, and usually it is openapi.yaml
fileName: ${specification.fileName:config/openapi.yaml}
# The content type of the specification file. In most cases, we are using yaml format.
contentType: ${specification.contentType:text/yaml}

```

You can overwrite the file location and filename as well as the content type in values.yml file. For example, some developers use openapi.json for the specification. 

The SpecDisplayHandler will ensure that a get request to /spec.yaml will return the openapi.ymal or openapi.json. 

##### FaviconHandler

This handler return the favicon.ico from the default module resources/config folder and this file can be overwrite in the externalized config folder. Most browsers will reuqest this file when loading an HTML file. This handler will prevent 404 error on the server. 

### Configuration

To enable the Swagger UI renderer, we need to add the following hanlders and paths to the handler.yml

```
handler.handlers:
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  - com.networknt.specification.FaviconHandler@favicon

paths:
  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui
  - path: '/favicon.ico'
    method: 'get'
    exec:
      - favicon


```

The other config file is the specification.yml and the content is here for the light-gateway and http-sidecar. 

```
---
# Specification type and file name definition
# The filename with path of the specification file, and usually it is openapi.yaml
fileName: ${specification.fileName:openapi.yaml}
# The content type of the specification file. In most cases, we are using yaml format.
contentType: ${specification.contentType:text/yaml}

```

### http-sidecar

The http-sidecar is one of the examples to use the Swagger UI and here is the steps you can follow to access the Swagger-UI.

```
cd ~/networknt
git clone https://github.com/networknt/http-sidecar.git
cd http-sidecar
mvn clean install
java -jar -Dlight-4j-config-dir=config/local target/http-sidecar.jar
```

Once the server is up and running. Open a browser and enter the following URL to acces the Swagger UI.

```
https://localhost:9445/specui.html
```

### light-gateway

The light-gateway is another examaples and here is the steps to follow. 

```
cd ~/networknt
git clone https://github.com/networknt/light-gateway.git
cd light-gateway
mvn clean install
java -jar -Dlight-4j-config-dir=config/local target/light-gateway.jar

```

Once the server is up and running. Open a browser and enter the following URL to acces the Swagger UI.

```
https://localhost:9445/specui.html
```
