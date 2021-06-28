---
title: "Kotlin Petstore Tutorial"
date: 2019-03-11T21:51:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Everybody knows that light-4j is a Java framework; however, @logi build an application purely in Kotlin on top of light-4j last year. I was surprised for the clean and concise test server and test cases from his [gists][], so I spent some time to learn Kotlin. The more I learn it, the more I love it. With the first demo application created, I created a [code generator for Kotlin][] based on the OpenAPI 3.0 specification.  

In this tutorial, we are going to walk through the generated Kotlin petstore application and compare it with the Java version. 

### Workspace

Before we start the tutorial, let's prepare the workspace by checking out several repositories from the GitHub. You first need to have a workspace folder created. I am using networknt under my home directory. 

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/light-example-kotlin.git
git clone https://github.com/networknt/model-config.git
```

### Build light-codegen

We are going to build the light-codegen and use the command line to generate the project. 

```
cd ~/networknt/light-codegen
mvn clean install
```

### Specification

We are going to use the same OpenAPI 3.0 petstore specification for the code generation. Also, the same config.json will be used to control the code generation. The specification openapi.yaml and config.json can be found in model-config repository at https://github.com/networknt/model-config/tree/master/rest/openapi/petstore/1.0.0

In the previous step, we have checked out the model-config respository to our workspace. You can browser the petstore specification and config on your local computer. 

### Generator

The generator for Kotlin is similar to the OpenAPI 3.0 generator as the inputs are the same. You only need to switch the framework parameter in the command line to openapikotlin instead of openapi. Please visit the reference document for the [OpenAPI Kotlin Generator][] for more details. 

To following command line can be used to generate a Kotlin project. 


```
cd ~/networknt
rm -rf light-example-kotlin/openapi/petstore
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapikotlin -o light-example-kotlin/openapi/petstore -m model-config/rest/openapi/petstore/1.0.0/openapi.yaml -c model-config/rest/openapi/petstore/1.0.0/config.json
```

As you can see that we removed the existing example in light-example-kotlin repository and regenerate it. If you want to see the version we generated, you can visit the repository on the GitHub at https://github.com/networknt/light-example-kotlin/tree/master/openapi/petstore


### Build and Test

With the above command lines, a new project was created in the light-example-kotlin/openapi folder called petstore. Let's build the project and start the server. 

```
cd ~/networknt/light-example-kotlin/openapi/petstore
./gradlew clean build run
```

Once the server is up and running, let's send the following request to the server to confirm it is working. 

```
curl -k https://localhost:8443/v1/pets/1
```

The result should be:

```
{"id":1,"name":"Jessica Right","tag":"pet"}
```

### Handlers

Let's shutdown the server with CTRL+C on the server terminal. Once the server is down, let's open a project and walkthough the code. 

First let's take a look at one of the handlers. 

```
package com.networknt.petstore.handler

import com.networknt.handler.LightHttpHandler
import io.undertow.server.HttpServerExchange
import io.undertow.util.HttpString

class PetsPetIdGetHandler : LightHttpHandler {
    
    @Throws(Exception::class)
    override fun handleRequest(exchange: HttpServerExchange) {
        exchange.responseHeaders.add(HttpString("Content-Type"), "application/json")
        exchange.responseSender.send("{\"id\":1,\"name\":\"Jessica Right\",\"tag\":\"pet\"}")
    }
}
```

It is very similar to the Java handler but with less code.

### Test Cases

Before looking at the handler test cases, let's take a look at the LightTestServer.kt

```
package com.networknt.petstore.handler

import assertk.Assert
import assertk.assertions.contains
import assertk.assertions.isEqualTo
import com.networknt.client.Http2Client
import com.networknt.server.Server
import io.undertow.UndertowOptions
import io.undertow.client.ClientCallback
import io.undertow.client.ClientExchange
import io.undertow.client.ClientRequest
import io.undertow.client.ClientResponse
import io.undertow.util.FlexBase64
import io.undertow.util.Headers
import io.undertow.util.HttpString
import io.undertow.util.Methods
import mu.KotlinLogging
import org.junit.jupiter.api.extension.AfterAllCallback
import org.junit.jupiter.api.extension.BeforeAllCallback
import org.junit.jupiter.api.extension.ExtensionContext
import org.xnio.OptionMap
import java.io.IOException
import java.net.ServerSocket
import java.net.URI
import java.util.*
import java.util.concurrent.CountDownLatch
import java.util.concurrent.atomic.AtomicReference

/**
 * Junit5 Extension which sets up a light-server BeforeAll tests and tears it down AfterAll.
 * Use with `@ExtendWith(LightTestServer::class)`
 *
 * The first time a server is started in a particular VM, a random port is assigned to it to avoid clashes between
 * concurrent test runs or other active servers.
 *
 * There are also static utility methods to make requests to the configured server.
 */
class LightTestServer() : BeforeAllCallback, AfterAllCallback {

    // EXTENSION LIFE-CYCLE METHODS

    var oldIsDynamicPort: Boolean? = null
    var oldHttpsPort: Int? = null

    // Patch server config and start server
    override fun beforeAll(context: ExtensionContext?) {
        oldIsDynamicPort = Server.getServerConfig().isDynamicPort
        oldHttpsPort = Server.getServerConfig().httpsPort
        Server.getServerConfig().isDynamicPort = false
        Server.getServerConfig().httpsPort = httpsPort
        Server.start()
    }

    // Stop server and unpatch config
    override fun afterAll(context: ExtensionContext?) {
        Server.stop()
        Server.getServerConfig().isDynamicPort = oldIsDynamicPort!!
        Server.getServerConfig().httpsPort = oldHttpsPort!!
    }

    companion object {

        val log = KotlinLogging.logger {}

        // SERVER STATE

        val httpsPort = randomFreePort(40000, 60000)
        val baseUrl = "https://localhost:$httpsPort"


        // MAKE REQUESTS TO SERVER

        /** Make a GET request to the server maintained by this extension. */
        fun makeGetRequest(path: String, auth: String? = null): ClientResponse {
            return makeRequest(path, Methods.GET, null, auth)
        }

        /** Make a POST request to the server maintained by this extension. */
        fun makePostRequest(path: String, body: String, auth: String? = null): ClientResponse {
            return makeRequest(path, Methods.POST, body, auth)
        }

        /** Make a PUT request to the server maintained by this extension. */
        fun makePutRequest(path: String, body: String, auth: String? = null): ClientResponse {
            return makeRequest(path, Methods.PUT, body, auth)
        }

        /** Make a DELETE request to the server maintained by this extension. */
        fun makeDeleteRequest(path: String, auth: String? = null): ClientResponse {
            return makeRequest(path, Methods.DELETE, null, auth)
        }

        /** Finds a random free port by attempting to listen on random ports until it succeeds. */
        fun randomFreePort(minPort: Int, maxPort: Int): Int {
            val random = Random()
            while (true) {
                val port = random.nextInt(maxPort - minPort) + minPort
                try {
                    val ss = ServerSocket(port)
                    ss.close()
                    return port
                } catch (e: IOException) {
                    log.info { "Port ${port} was busy" }
                }
            }
        }

        /** Make a request to the server maintained by this extension. */
        fun makeRequest(path: String, method: HttpString, body: String?, auth: String? = null): ClientResponse {
            log.info { "${method} :: $baseUrl :: ${path}" }

            val client = Http2Client.getInstance()

            client.connect(
                URI(baseUrl),
                Http2Client.WORKER,
                Http2Client.SSL,
                Http2Client.BUFFER_POOL,
                OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)
            ).get().use { connection ->

                val request = ClientRequest().setPath(path).setMethod(method)
                authenticate(request, auth)
                val latch = CountDownLatch(1)
                val reference = AtomicReference<ClientResponse>()
                val callback: ClientCallback<ClientExchange>

                if (body == null) {
                    callback = client.createClientCallback(reference, latch)
                } else {
                    log.info { "body: ${body}" }
                    val firstChar = if (body.length > 0) body[0] else '\u0000'
                    if (firstChar == '[' || firstChar == '{') {
                        request.requestHeaders.put(Headers.CONTENT_TYPE, "application/json")
                    } else {
                        request.requestHeaders.put(Headers.CONTENT_TYPE, "text/plain")
                    }
                    request.requestHeaders.put(Headers.TRANSFER_ENCODING, "chunked")
                    callback = client.createClientCallback(reference, latch, body)
                }

                connection.sendRequest(request, callback)
                latch.await()

                val response = reference.get()
                log.info { "Response code = ${response.responseCode}" }
                log.info { "Response body = ${response.getAttachment(Http2Client.RESPONSE_BODY)}" }
                return response
            }

        }

        private fun authenticate(request: ClientRequest, auth: String?) {
            if (auth == null) return

            log.info { "auth = ${auth}" }
            val bytes = auth.toByteArray(Charsets.UTF_8)
            log.info { "bytes = ${bytes}" }
            val encoded = FlexBase64.encodeString(bytes, false)
            log.info { "encoded = ${encoded}" }
            request.requestHeaders.add(
                Headers.AUTHORIZATION,
                "Basic ${encoded}"
            )
        }
    }
}

fun Assert<ClientResponse>.rcIsEqualTo(expected: Int) = given { actual ->
    assertThat(actual.responseCode, "Response Code").isEqualTo(expected)
}

fun Assert<ClientResponse>.bodyIsEqualTo(expected: String) = given { actual ->
    assertThat(actual.getAttachment(com.networknt.client.Http2Client.RESPONSE_BODY), "Body").isEqualTo(expected)
}

fun Assert<ClientResponse>.bodyContains(expected: String)  = given { actual ->
    assertThat(actual.getAttachment(com.networknt.client.Http2Client.RESPONSE_BODY), "Body").contains(expected)
}

```

This class contains all the functions that support handler test cases. It has more functions then the Java implementation so that the test case will be significantly reduced in size. Let's take a look at one of the generated test cases. 

```
package com.networknt.petstore.handler
import assertk.all
import assertk.assertThat
import mu.KotlinLogging
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.extension.ExtendWith


@ExtendWith(LightTestServer::class)
class PetsPetIdGetHandlerTest {
    companion object {
        val log = KotlinLogging.logger {}
    }

    @Test
    fun `test PetsPetIdGetHandlerTest() PetsPetIdGetHandler success` () {
        /*
        
        val response = LightTestServer.makeGetRequest("/v1/pets/LCCUecwwemS")
        
        assertThat(response).all {
            rcIsEqualTo(200)
            bodyContains("any string from the body to be replaced")
        }
        */
    }
}


```

As you see that we have commented out the assertion code for the test cases. To show users how to write real test cases, we have created a new project called petstore-unittest located at https://github.com/networknt/light-example-kotlin/tree/master/openapi/petstore-unittest

Let's take a look at a fully implemented test case. 

```
package com.networknt.petstore.handler
import assertk.all
import assertk.assertThat
import mu.KotlinLogging
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.extension.ExtendWith


@ExtendWith(LightTestServer::class)
class PetsPetIdGetHandlerTest {
    companion object {
        val log = KotlinLogging.logger {}
    }

    @Test
    fun `test PetsPetIdGetHandlerTest() PetsPetIdGetHandler success` () {
        val response = LightTestServer.makeGetRequest("/v1/pets/hhRKMxyUaDKvRpYwcXmLHu")
        
        assertThat(response).all {
            rcIsEqualTo(200)
            bodyContains("id")
        }
    }
}
```

As you can see that it is very simple to send a request to the server and assert the response from the server. 



[gists]: https://gist.github.com/logi?direction=asc&sort=created
[code generator for Kotlin]: /tool/light-codegen/openapi-kotlin-generator/


