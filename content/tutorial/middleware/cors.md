---
title: "CORS - Cross-Origin Resource Sharing"
date: 2017-11-06T17:45:44-05:00
categories: []
keywords: [tutorial]
weight: 10
aliases: []
toc: false
draft: false
---


For some of the APIs/services, the endpoints will be accessed from a Single Page 
Application(React/Vue/Angular) served from another domain. In this case, the API
server needs to handle the pre-flight OPTIONS request to enable CORS. As light*4j
platform can serve single page application from the same domain as the API/service,
normally you don't need to do that if you are the owner of both client and service.
However, during the development of the single page application, most developers will
use nodejs to serve the Javascript with webpack hot reload to increase productivity
and you can use a proxy on the nodejs server to your APIs/services built on top of
the light*4j frameworks. In this setup, you need to have CORS to be enabled on the
API server so that the Javascript single page app can call the API from browser. 

As CORS only used in above scenario, the handler is not wired in by default in
light-codegen. 

For this CORS API, I have created both Swagger 2.0 and OpenAPI 3.0 specifications
and light-codegen config file. 

As we are moving to OpenAPI 3.0 specification, the Swagger 2.0 specification is just
kept as a reference. In this tutorial, we are going to use OpenAPI 3.0 spec only.

If you are forced to stay with Swagger 2.0 specification, you can follow this tutorial
to generate the code using Swagger spec and all additional updates are valid for both.
 

* [CORS API Swagger 2.0 Spec][] 
* [CORS API OpenAPI 3.0 Spec][]


### OpenAPI 3.0 Specification

Here is the openapi.yaml file and you can see what we  have three demo endpoints for
get, post and put.

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

Given that we have the specification ready, we need to create a config.json file to control
how light-codegen can generate the project. 

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

With both specification and config.json available, let's scaffold the project. Please note
that we have another file called openapi.json and it is the input for the light-codegen. The
YAML format is easy for human to read but JSON is easy to be consumed by computers. The two
formats are inter-changeable in swagger editor and we have both in the github model-config repo.

Assume that you have a working directory called networknt under your user home directory. Before
we generate the project, we are going to backup the existing folder in light-example-4j and we
are going to generate the project to the cors folder. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install -DskipTests
mv light-example-4j/rest/openapi/cors light-example-4j/rest/openapi/cors.bak
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapi -o light-example-4j/rest/openapi/cors -m model-config/rest/openapi/cors/openapi.json -c model-config/rest/openapi/cors/config.json

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

Let's send a pre-flight options request and see what is the response. Remember that by
default, the CorsHandler is not plugged in yet. 

```
curl -k -H "Origin: http://example.com" -H "Access-Control-Request-Method: POST"  -H "Access-Control-Request-Headers: X-Requested-With"  -X OPTIONS --verbose https://localhost:8443/v1/postData
```

The response is something like this.

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
< HTTP/1.1 405 Method Not Allowed
< Server: L
< Content-Length: 127
< Date: Sun, 10 Dec 2017 20:39:27 GMT
< 
* Connection #0 to host localhost left intact
{"statusCode":405,"code":"ERR10008","message":"METHOD_NOT_ALLOWED","description":"Request method cannot be found in the spec."}

```

Please note that 405 Method Not Allowed is returned which indicates CORS is not supported
on this server for POST request. You can test PUT request by yourself. 

### Enable CORS

In this step, let's update the generated project to enable CORS. 

If you want to limit only several domains for CORS, you also need to create cors.yml
in config folder.

And here is an example of cors.yml

```
enabled: true
allowedOrigins:
- https://localhost
allowedMethods:
- GET
- POST
```

Let's put this file in src/main/resources/config folder.

To enable CORS support, you need to add cors module in pom.xml, you need to update 
pom.xml to add a dependency.

```
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>cors</artifactId>
            <version>${version.light-4j}</version>
        </dependency>

```

Now, we need to wire in the CorsHandler in service.yml file in the same config folder.


Note that cors handler is in front of swagger so that pre-flight OPTIONS request will be 
returned before swagger validation is done as OPTIONS methods are not defined in OpenAPI 
specification. 

Here is the service.yml after adding CorsHttpHandler

```yaml
# Singleton service factory configuration/IoC injection
singletons:
# OpenAPI parser validator implementation
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Callback>:
  - com.networknt.oas.validator.impl.CallbackValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Contact>:
  - com.networknt.oas.validator.impl.ContactValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.EncodingProperty>:
  - com.networknt.oas.validator.impl.EncodingPropertyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Example>:
  - com.networknt.oas.validator.impl.ExampleValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ExternalDocs>:
  - com.networknt.oas.validator.impl.ExternalDocsValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Header>:
  - com.networknt.oas.validator.impl.HeaderValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.License>:
  - com.networknt.oas.validator.impl.LicenseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Info>:
  - com.networknt.oas.validator.impl.InfoValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Link>:
  - com.networknt.oas.validator.impl.LinkValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Schema>:
  - com.networknt.oas.validator.impl.SchemaValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.MediaType>:
  - com.networknt.oas.validator.impl.MediaTypeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OAuthFlow>:
  - com.networknt.oas.validator.impl.OAuthFlowValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.OpenApi3>:
  - com.networknt.oas.validator.impl.OpenApi3Validator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.RequestBody>:
  - com.networknt.oas.validator.impl.RequestBodyValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Operation>:
  - com.networknt.oas.validator.impl.OperationValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Parameter>:
  - com.networknt.oas.validator.impl.ParameterValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Path>:
  - com.networknt.oas.validator.impl.PathValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Response>:
  - com.networknt.oas.validator.impl.ResponseValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityRequirement>:
  - com.networknt.oas.validator.impl.SecurityRequirementValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.SecurityScheme>:
  - com.networknt.oas.validator.impl.SecuritySchemeValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Server>:
  - com.networknt.oas.validator.impl.ServerValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.ServerVariable>:
  - com.networknt.oas.validator.impl.ServerVariableValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Tag>:
  - com.networknt.oas.validator.impl.TagValidator
- com.networknt.oas.validator.Validator<com.networknt.oas.model.Xml>:
  - com.networknt.oas.validator.impl.XmlValidator
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
  - com.networknt.cors.PathHandlerProvider
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test2StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  #Cors handler to handler post/put pre-flight
  - com.networknt.cors.CorsHttpHandler
  # Parsing OpenAPI 3.0 specification based on request uri and method.
  - com.networknt.openapi.OpenApiHandler
  # Security JWT token verification and scope verification (depending on OpenApiHandler)
  - com.networknt.openapi.JwtVerifyHandler
  # Body Parse body based on content type in the header.
  - com.networknt.body.BodyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Sanitizer Encode cross site scripting
  - com.networknt.sanitizer.SanitizerHandler
  # Validator Validate request based on OpenAPI 3.0 specification (depending on OpenApiHandler and BodyHandler)
  - com.networknt.openapi.ValidatorHandler
```

### Build and Test again

Now let's start the server again. Please make sure that you kill the previous server with Ctrl+C
on the terminal. 

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
