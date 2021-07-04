---
title: "CORS - Cross-Origin Resource Sharing"
date: 2017-11-06T17:45:44-05:00
categories: []
keywords: [tutorial]
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---


For some APIs/services, the endpoints will be accessed from a Single Page Application(React/Vue/Angular) served from another domain. In this case, the API server needs to handle the preflight OPTIONS request to enable CORS. As the light-4j platform can serve a single page application from the same domain as the API/service, you normally don't need to do that if you are the owner of both client and service.
However, during the development of the single page application, most developers will use nodejs to serve the Javascript with webpack hot reload to increase productivity, and you can use a proxy on the nodejs server to your APIs/services built on top of the light-4j frameworks. In this setup, you need to have CORS to be enabled on the API server so that the Javascript single page app can call the API from browser. 

As CORS only used in the above scenario, the handler is not wired in by default in light-codegen scaffolded projects. 

For this CORS API example, I have created both Swagger 2.0 and OpenAPI 3.0 specifications and light-codegen config file. 

As we are moving to the OpenAPI 3.0 specification, the Swagger 2.0 specification is just kept as a reference. In this tutorial, we are going to use OpenAPI 3.0 spec only.

If you are forced to stay with Swagger 2.0 specification, you can follow this tutorial to generate the code using Swagger spec and all additional updates are valid for both.
 

* [CORS API Swagger 2.0 Spec][] 
* [CORS API OpenAPI 3.0 Spec][]


### OpenAPI 3.0 Specification

Here is the openapi.yaml file and you can see we have three demo endpoints for `get`, `post` and `put`.

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: CORS Demo
  license:
    name: MIT
servers:
  - url: 'https://cors.lightapi.net/v1'
paths:
  /getData:
    get:
      description: Return an array of strings
      operationId: getData
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/KeyValue'
      security:
        - cors_auth:
            - cors.r
  /postData:
    post:
      description: Return an array of strings
      operationId: postData
      requestBody:
        description: Key value pair
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/KeyValue'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/KeyValue'
      security:
        - cors_auth:
            - cors.r
            - cors.w
  /putData:
    post:
      description: Return an array of strings
      operationId: putData
      requestBody:
        description: Key value pair
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/KeyValue'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/KeyValue'
      security:
        - cors_auth:
            - cors.r
            - cors.w
components:
  securitySchemes:
    cors_auth:
      type: oauth2
      description: This API uses OAuth 2 with the client credential grant flow.
      flows:
        clientCredentials:
          tokenUrl: 'https://localhost:6882/token'
          scopes:
            cors.w: write
            cors.r: read
  schemas:
    KeyValue:
      type: object
      required:
        - key
        - value
      properties:
        key:
          type: string
          description: key
        value:
          type: string
          description: value
```


### Scaffold CORS API

Given that we have the specification ready, we need to create a config.json file to control how light-codegen can generate the project. 

Here is the final config.json that you can find from the above specification link. 

```json
{
  "name": "cors",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "cors",
  "rootPackage": "com.networknt.cors",
  "handlerPackage":"com.networknt.cors.handler",
  "modelPackage":"com.networknt.cors.model",
  "overwriteHandler": true,
  "overwriteHandlerTest": true,
  "overwriteModel": true,
  "httpPort": 8080,
  "enableHttp": false,
  "httpsPort": 8443,
  "enableHttps": true,
  "enableHttp2": true,
  "enableRegistry": false,
  "supportDb": false,
  "supportH2ForTest": false,
  "supportClient": false
}

```

With both the specification file openapi.yaml and config.json available, let's scaffold the project. Please note that we have another file called openapi.json and it is another optional input for the light-codegen. The YAML format is preferred as it is easy for a human to read and easy to be consumed by computers. The two formats are interchangeable in the swagger editor, and we have both in the GitHub model-config repo. 

Assume that you have a working directory called networknt under your user home directory. Before we generate the project, we are going to back up the existing folder in light-example-4j, and we are going to generate the project to the cors folder. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install -DskipTests
mv light-example-4j/rest/openapi/cors light-example-4j/rest/openapi/cors.bak
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/cors -m model-config/rest/openapi/cors/openapi.yaml -c model-config/rest/openapi/cors/config.json

``` 

### Build and Test

Now let's build and start the generated project and make sure it is working. 

```
cd ~/networknt/light-example-4j/rest/openapi/cors
mvn clean install exec:exec
```

Now let's send a get request to the server. 

```
curl -k https://localhost:8443/v1/getData

```

And here is the sample response from the specification. 

```json
[{"key":"key1","value":"value1"},{"key":"key2","value":"value2"}]
```

Now let's send a post request to the server. 

```
curl -k -X POST https://localhost:8443/v1/postData -H 'content-type: application/json' -d '{"key":"key1","value": "value1"}'
```

And you will see the following response.

```json
{"key":"key1","value":"value1"}
```

Above API access are just regular access to the endpoint defined in the specification. They are working as we have example response defined in the spec. Next, let's try to send an options request to the server to one of the endpoints. 

Let's send a pre-flight options request and see what the response is. Remember that by default, the CorsHandler is not plugged in yet. 

```
curl -k -H "Origin: http://example.com" -H "Access-Control-Request-Method: POST"  -H "Access-Control-Request-Headers: X-Requested-With"  -X OPTIONS --verbose https://localhost:8443/v1/postData
```

The response is something like this.

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  start date: Jan 19 14:27:52 2013 GMT
*  expire date: Jan 17 14:27:52 2023 GMT
*  issuer: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55fa7a9c5580)
> OPTIONS /v1/postData HTTP/2
> Host: localhost:8443
> User-Agent: curl/7.58.0
> Accept: */*
> Origin: http://example.com
> Access-Control-Request-Method: POST
> Access-Control-Request-Headers: X-Requested-With
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 404 
< server: L
< content-type: application/json
< content-length: 147
< date: Sat, 21 Sep 2019 22:52:31 GMT
< 
* Connection #0 to host localhost left intact
{"statusCode":404,"code":"ERR10048","message":"PATH_NOT_IMPLEMENTED","description":"No handler defined for path '/v1/postData'","severity":"ERROR"}
```

Please note that 404 PATH_NOT_IMPLEMENTED is returned which indicates there is no handler defined in the handler.yml for the options request. 

### Enable CORS

In this step, let's update the generated project to enable CORS. 

If you want to limit only several domains for CORS, you also need to create cors.yml in config folder.

And here is an example of cors.yml

```
enabled: true
allowedOrigins:
- https://localhost
allowedMethods:
- GET
- POST
- PUT
```

Let's put this file in src/main/resources/config folder.

To enable CORS support, you need to add cors module in pom.xml, you need to update pom.xml to add a dependency.

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>cors</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

Now, we need to wire in the CorsHttpHandler in handler.yml file in the same config folder.


Note that cors handler is in front of OpenAPI so that pre-flight OPTIONS request will be returned before openapir validation is done as OPTIONS methods are not defined in OpenAPI specification. 


Here is the handler.yml after adding CorsHttpHandler. 


```yaml
# Handler middleware chain configuration
#----------------------------------------
enabled: true

# Configuration for the LightHttpHandler. The handler is the base class  for all middleware, server and health handlers
# set the Status Object in the AUDIT_INFO, for auditing purposes
# default, if not set:false
auditOnError: ${handler.auditOnError:false}

# set the StackTrace in the AUDIT_INFO, for auditing purposes
# default, if not set:false
auditStackTrace: ${handler.auditStackTrace:false}

#------------------------------------------------------------------------------
# Support individual handler chains for each separate endpoint. It allows framework
# handlers like health check, server info to bypass majority of the middleware handlers
# and allows mixing multiple frameworks like OpenAPI and GraphQL in the same instance.
#
# handlers  --  list of handlers to be used across chains in this microservice
#               including the routing handlers for ALL endpoints
#           --  format: fully qualified handler class name@optional:given name
# chains    --  allows forming of [1..N] chains, which could be wholly or
#               used to form handler chains for each endpoint
#               ex.: default chain below, reused partially across multiple endpoints
# paths     --  list all the paths to be used for routing within the microservice
#           ----  path: the URI for the endpoint (ex.: path: '/v1/pets')
#           ----  method: the operation in use (ex.: 'post')
#           ----  exec: handlers to be executed -- this element forms the list and
#                       the order of execution for the handlers
#
# IMPORTANT NOTES:
# - to avoid executing a handler, it has to be removed/commented out in the chain
#   or change the enabled:boolean to false for a middleware handler configuration.
# - all handlers, routing handler included, are to be listed in the execution chain
# - for consistency, give a name to each handler; it is easier to refer to a name
#   vs a fully qualified class name and is more elegant
# - you can list in chains the fully qualified handler class names, and avoid using the
#   handlers element altogether
#------------------------------------------------------------------------------
handlers:
  # Light-framework cross-cutting concerns implemented in the microservice
  - com.networknt.exception.ExceptionHandler@exception
  - com.networknt.metrics.MetricsHandler@metrics
  - com.networknt.traceability.TraceabilityHandler@traceability
  - com.networknt.correlation.CorrelationHandler@correlation
  - com.networknt.openapi.OpenApiHandler@specification
  - com.networknt.openapi.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  - com.networknt.cors.CorsHttpHandler@cors
  # DumpHandler is to dump detail request/response info to log, useful for troubleshooting but not suggested to use in production due to it may lower the performance
  # - com.networknt.dump.DumpHandler@dump
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.openapi.ValidatorHandler@validator
  # Customer business domain specific cross-cutting concerns handlers
  # - com.example.validator.CustomizedValidator@custvalidator
  # Framework endpoint handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.networknt.specification.SpecDisplayHandler@spec
  - com.networknt.specification.SpecSwaggerUIHandler@swaggerui
  # - com.networknt.metrics.prometheus.PrometheusGetHandler@getprometheus
  # Business Handlers
  - com.networknt.cors.handler.GetDataGetHandler
  - com.networknt.cors.handler.PostDataPostHandler
  - com.networknt.cors.handler.PutDataPostHandler


chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - cors
    - specification
    - security
    - body
    - audit
#    - dump
    - sanitizer
    - validator

paths:
  - path: '/v1/getData'
    method: 'options'
    exec:
      - cors
  - path: '/v1/getData'
    method: 'GET'
    exec:
      - default
      - com.networknt.cors.handler.GetDataGetHandler
  - path: '/v1/postData'
    method: 'options'
    exec:
      - cors
  - path: '/v1/postData'
    method: 'POST'
    exec:
      - default
      - com.networknt.cors.handler.PostDataPostHandler
  - path: '/v1/putData'
    method: 'options'
    exec:
      - cors
  - path: '/v1/putData'
    method: 'POST'
    exec:
      - default
      - com.networknt.cors.handler.PutDataPostHandler

  - path: '/health/com.networknt.cors-1.0.1'
    method: 'get'
    exec:
      - health

  - path: '/server/info'
    method: 'get'
    exec:
      - info


  - path: '/spec.yaml'
    method: 'get'
    exec:
      - spec
  - path: '/specui.html'
    method: 'get'
    exec:
      - swaggerui

```

Please note that the following line has been added to the handlers section and the `cors` has been added to the default chain right after the correlation. 

```
- com.networknt.cors.CorsHttpHandler@cors
```

As the options method is not defined in the openapi.yaml, we need to add this method to each endpoint and the handler is configured to `cors`. 


### Build and Test again

Now let's start the server again. Please make sure that you kill the previous server with Ctrl+C on the terminal. 

```
cd ~/networknt/light-example-4j/rest/openapi/cors
mvn clean install exec:exec
```

Test the pre-flight OPTIONS again with the same curl command. 

```
curl -k -H "Origin: http://example.com" -H "Access-Control-Request-Method: POST"  -H "Access-Control-Request-Headers: X-Requested-With"  -X OPTIONS --verbose https://localhost:8443/v1/postData
```

And the result
```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8443 (#0)
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  start date: Jan 19 14:27:52 2013 GMT
*  expire date: Jan 17 14:27:52 2023 GMT
*  issuer: C=GB; ST=State; L=City; O=Org; OU=OU; CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
> OPTIONS /v1/postData HTTP/1.1
> Host: localhost:8443
> User-Agent: curl/7.55.1
> Accept: */*
> Origin: http://example.com
> Access-Control-Request-Method: POST
> Access-Control-Request-Headers: X-Requested-With
> 
< HTTP/1.1 200 OK
< Access-Control-Allow-Headers: X-Requested-With
< Server: L
< Access-Control-Allow-Credentials: true
< Content-Length: 0
< Access-Control-Allow-Methods: GET
< Access-Control-Allow-Methods: POST
< Access-Control-Max-Age: 3600
< Date: Sun, 10 Dec 2017 21:04:56 GMT
< 
* Connection #0 to host localhost left intact

```

As you can see that "Access-Control-Allow-Methods: POST" in the response. 

The source code for the cors example can be found at https://github.com/networknt/light-example-4j/tree/master/rest/openapi/cors


[CORS API Swagger 2.0 Spec]: https://github.com/networknt/model-config/tree/master/rest/swagger/cors
[CORS API OpenAPI 3.0 Spec]: https://github.com/networknt/model-config/tree/master/rest/openapi/cors
