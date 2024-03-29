---
title: "Handler"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

An interface for middleware handlers. All middleware handlers must implement this interface so that the handler can be plugged into the request/response chain during server startup with SPI (Service Provider Interface). The entire light-4j framework is a core server that provides a plugin structure for hooking up all sorts of plugins to handle different cross-cutting concerns.

The middleware handlers are designed based on the chain of responsibility pattern.


### MiddlewareHandler
```
/**
 * A interface for middleware handlers. All middleware handlers must implement this interface
 * so that the handler can be plugged in to the request/response chain during server startup
 * with SPI (Service Provider Interface). The entire light-4j framework is a core server that
 * provides a plugin structure to hookup all sorts of plugins to handler different cross-cutting
 * concerns.
 *
 * The middleware handlers are designed based on chain of responsibility pattern.
 *
 * This handler extends LightHttpHandler which has a default method to handle the error status
 * response.
 *
 * @author Steve Hu
 */
public interface MiddlewareHandler extends LightHttpHandler {
    /**
     * Get the next handler in the chain
     *
     * @return HttpHandler
     */
    HttpHandler getNext();

    /**
     * Set the next handler in the chain
     *
     * @param next HttpHandler
     * @return MiddlewareHandler
     */
    MiddlewareHandler setNext(final HttpHandler next);

    /**
     * Indicate if this handler is enabled or not.
     *
     * @return boolean true if enabled
     */
    boolean isEnabled();

    /**
     * Register this handler to the handler registration.
     */
    void register();
}

```

To add/remove/replace middleware handlers/plugins, update the service.yml config file to bind them with the MiddlewareHandler interface. These handlers/plugins will be executed in sequence for every incoming request.

Here is the default middleware configuration generated by [light-codegen][] for RESTful API built with OpenAPI 3.0 specification. 

```yaml
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
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

### MiddlewareHandler Implementations

There are example implementations in [middleware][]

### LightHttpHandler

This is an interface that extends from the Undertow HttpHandler, and it provides the status error handling in the framework. It is recommended to implement this interface for your business handlers. If you are using the light-codegen to generate the project, all business handlers will implement this interface by default. 

```
package com.networknt.handler;

import com.networknt.status.Status;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Headers;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * This is an extension of HttpHandler that provides a default method to handle
 * error status. All API handler should extend from this interface.
 *
 * @author Steve Hu
 */
public interface LightHttpHandler extends HttpHandler {
    Logger logger = LoggerFactory.getLogger(LightHttpHandler.class);
    String ERROR_NOT_DEFINED = "ERR10042";

    default void setExchangeStatus(HttpServerExchange exchange, String code, final Object... args) {
        Status status = new Status(code, args);
        if(status.getStatusCode() == 0) {
            // There is no entry in status.yml for this particular error code.
            status = new Status(ERROR_NOT_DEFINED, code);
        }
        exchange.setStatusCode(status.getStatusCode());
        exchange.getResponseHeaders().put(Headers.CONTENT_TYPE, "application/json");
        exchange.getResponseSender().send(status.toString());
        StackTraceElement[] elements = Thread.currentThread().getStackTrace();
        logger.error(status.toString() + " at " + elements[2].getClassName() + "." + elements[2].getMethodName() + "(" + elements[2].getFileName() + ":" + elements[2].getLineNumber() + ")");
    }
}

```

### HandlerProvider

Handler provider is an interface, and every project must have an implementation for this interface with all the endpoint path/method pairs map to LightHttpHandler implementations. The implementation of this interface is defined in the service.yml config file. 

```
/*
 * Copyright (c) 2016 Network New Technologies Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * You may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.networknt.handler;

import io.undertow.server.HttpHandler;

/**
 * A user handler provider interface. The framework has some middleware handlers and
 * these are wired into the request/response chain at the right sequence. At the end
 * of the request chain, the user business logic need to be called to do the real
 * processing and it is usually implemented as a serial of handlers. These handlers
 * needs to be grouped together and mapped to certain URLs and http methods.
 *
 * The mapping class implements this HandlerProvider so that it can be loaded during
 * server startup to inject into the handler chain.
 *
 * @author Steve Hu
 */
public interface HandlerProvider {
    /**
     * Every handler provider needs to implement this method to return a HttpHanlder or
     * a chain of HttpHandlers.
     *
     * @return HttpHandler
     */
    HttpHandler getHandler();
}

```

### OrchestrationHandler

This is a handler instance that orchestrates multiple handler chains for the exchange.

```
package com.networknt.handler;

import io.undertow.server.HttpServerExchange;

/**
 * @author Nicholas Azar
 */
public class OrchestrationHandler implements LightHttpHandler {

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (Handler.start(exchange)) {
            Handler.next(exchange);
        } else {
            String methodPath = String.format("%s %s", exchange.getRequestMethod(), exchange.getRequestPath());
            setExchangeStatus(exchange, "ERR10048", methodPath);
        }
    }
}

```

### Handler

This is the class that handles the multiple chains of middleware handlers. 

```
package com.networknt.handler;

import com.networknt.utility.Tuple;
import com.networknt.config.Config;
import com.networknt.handler.config.HandlerConfig;
import com.networknt.handler.config.PathChain;
import com.networknt.service.ServiceUtil;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.AttachmentKey;
import io.undertow.util.HttpString;
import io.undertow.util.PathTemplateMatcher;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Random;

/**
 * @author Nicholas Azar
 */
public class Handler {

    private static final AttachmentKey<Integer> CHAIN_SEQ = AttachmentKey.create(Integer.class);
    private static final AttachmentKey<String> CHAIN_ID = AttachmentKey.create(String.class);
    private static final Logger logger = LoggerFactory.getLogger(Handler.class);
    private static final String CONFIG_NAME = "handler";
    // Accessed directly.
    public
    static HandlerConfig config = HandlerConfig.load();

    // each handler keyed by a name.
    static final Map<String, HttpHandler> handlers = new HashMap<>();
    static final Map<String, List<HttpHandler>> handlerListById = new HashMap<>();
    static final Map<HttpString, PathTemplateMatcher<String>> methodToMatcherMap = new HashMap<>();

    static {
        try {
            initHandlers();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        initPaths();
    }

    /**
     * Handle the next request in the chain.
     *
     * @param httpServerExchange The current requests server exchange.
     * @throws Exception Propagated exception in the handleRequest chain.
     */
    public static void next(HttpServerExchange httpServerExchange) throws Exception {
        HttpHandler httpHandler = getNext(httpServerExchange);
        if (httpHandler != null) {
            httpHandler.handleRequest(httpServerExchange);
        }
    }

    /**
     * Go to the next handler if the given next is none null.
     * Reason for this is for middleware to provide their instance next if it exists. Since if it exists,
     * the server hasn't been able to find the handler.yml.
     *
     * @param httpServerExchange The current requests server exchange.
     * @param next The next HttpHandler to go to if it's not null.
     * @throws Exception
     */
    public static void next(HttpServerExchange httpServerExchange, HttpHandler next) throws Exception {
        if (next != null) {
            next.handleRequest(httpServerExchange);
        } else {
            next(httpServerExchange);
        }
    }

    /**
     * Allow nexting directly to a flow.
     *
     * @param httpServerExchange The current requests server exchange.
     * @param execName The name of the next executable to go to, ie chain or handler. Chain resolved first.
     * @param returnToOrigFlow True if you want to call the next handler defined in your original chain after the provided execName is completed. False otherwise.
     * @throws Exception
     */
    public static void next(HttpServerExchange httpServerExchange, String execName, Boolean returnToOrigFlow) throws Exception {
        String currentChainId = httpServerExchange.getAttachment(CHAIN_ID);
        Integer currentNextIndex = httpServerExchange.getAttachment(CHAIN_SEQ);

        httpServerExchange.putAttachment(CHAIN_ID, execName);
        httpServerExchange.putAttachment(CHAIN_SEQ, 0);

        next(httpServerExchange);

        // return to current flow.
        if (returnToOrigFlow) {
            httpServerExchange.putAttachment(CHAIN_ID, currentChainId);
            httpServerExchange.putAttachment(CHAIN_SEQ, currentNextIndex);
            next(httpServerExchange);
        }
    }

    /**
     * Returns the instance of the next handler, rather then calling handleRequest on it.
     *
     * @param httpServerExchange The current requests server exchange.
     * @return The HttpHandler that should be executed next.
     */
    public static HttpHandler getNext(HttpServerExchange httpServerExchange) {
        String chainId = httpServerExchange.getAttachment(CHAIN_ID);
        List<HttpHandler> handlersForId = handlerListById.get(chainId);
        Integer nextIndex = httpServerExchange.getAttachment(CHAIN_SEQ);
        // Check if we've reached the end of the chain.
        if (nextIndex < handlersForId.size()) {
            httpServerExchange.putAttachment(CHAIN_SEQ, nextIndex + 1);
            return handlersForId.get(nextIndex);
        }
        return null;
    }

    /**
     * Returns the instance of the next handler, or the given next param if it's not null.
     *
     * @param httpServerExchange The current requests server exchange.
     * @param next If not null, return this.
     * @return The next handler in the chain, or next if it's not null.
     * @throws Exception
     */
    public static HttpHandler getNext(HttpServerExchange httpServerExchange, HttpHandler next) throws Exception {
        if (next != null) {
            return next;
        }
        return getNext(httpServerExchange);
    }



    /**
     * On the first step of the request, match the request against the configured paths. If the match is successful,
     * store the chain id within the exchange. Otherwise return false.
     *
     * @param httpServerExchange The current requests server exchange.
     * @return true if a handler has been defined for the given path.
     */
    public static boolean start(HttpServerExchange httpServerExchange) {
        // Get the matcher corresponding to the current request type.
        PathTemplateMatcher<String> pathTemplateMatcher = methodToMatcherMap.get(httpServerExchange.getRequestMethod());
        if (pathTemplateMatcher != null) {
            // Match the current request path to the configured paths.
            PathTemplateMatcher.PathMatchResult<String> result = pathTemplateMatcher.match(httpServerExchange.getRequestPath());
            if (result != null) {
                // Found a match, configure and return true;
                String id = result.getValue();
                httpServerExchange.putAttachment(CHAIN_ID, id);
                httpServerExchange.putAttachment(CHAIN_SEQ, 0);
                return true;
            }
        }
        return false;
    }

    /**
     * Build "handlerListById" and "reqTypeMatcherMap" from the paths in the config.
     */
    static void initPaths() {
        if (config != null && config.getPaths() != null) {
            for (PathChain pathChain : config.getPaths()) {
                HttpString method = new HttpString(pathChain.getMethod());

                // Use a random integer as the id for a given path.
                Integer randInt = new Random().nextInt();
                while (handlerListById.containsKey(randInt.toString())) {
                    randInt = new Random().nextInt();
                }

                // Flatten out the execution list from a mix of middleware chains and handlers.
                List<HttpHandler> handlers = getHandlersFromExecList(pathChain.getExec());
                if (handlers.size() > 0) {
                    // If a matcher already exists for the given type, at to that instead of creating a new one.
                    PathTemplateMatcher<String> pathTemplateMatcher = methodToMatcherMap.containsKey(method) ? methodToMatcherMap.get(method) : new PathTemplateMatcher<>();
                    pathTemplateMatcher.add(pathChain.getPath(), randInt.toString());
                    methodToMatcherMap.put(method, pathTemplateMatcher);
                    handlerListById.put(randInt.toString(), handlers);
                }
            }
        }
    }

    /**
     * Converts the list of chains and handlers to a flat list of handlers.
     * If a chain is named the same as a handler, the chain is resolved first.
     *
     * @param execs The list of names of chains and handlers.
     * @return A list containing references to the instantiated handlers
     */
    private static List<HttpHandler> getHandlersFromExecList(List<String> execs) {
        List<HttpHandler> handlersFromExecList = new ArrayList<>();
        if (execs != null) {
            for (String exec : execs) {
                handlersFromExecList.addAll(handlerListById.get(exec));
            }
        }
        return handlersFromExecList;
    }

    /**
     * Construct the named map of handlers.
     * @throws Exception
     */
    static void initHandlers() throws Exception {
        if (config != null && config.getHandlers() != null) {
            for (Object handler : config.getHandlers()) {
                // If the handler is configured as just a string, it's a fully qualified class name with a default constructor.
                if (handler instanceof String) {
                    Tuple<String, Class> namedClass = splitClassAndName((String) handler);
                    HttpHandler httpHandler = (HttpHandler) namedClass.second.newInstance();
                    handlers.put(namedClass.first, httpHandler);
                    handlerListById.put(namedClass.first, Collections.singletonList(httpHandler));
                } else if (handler instanceof Map) {
                    // If the handler is a map, the keys are the class name, values are the parameters.
                    for (Map.Entry<String, Object> entry : ((Map<String, Object>) handler).entrySet()) {
                        Tuple<String, Class> namedClass = splitClassAndName(entry.getKey());
                        // If the values in the config are a map, construct the object using named parameters.
                        if (entry.getValue() instanceof Map) {
                            HttpHandler httpHandler = (HttpHandler) ServiceUtil.constructByNamedParams(namedClass.second, (Map) entry.getValue());
                            handlers.put(namedClass.first, httpHandler);
                            handlerListById.put(namedClass.first, Collections.singletonList(httpHandler));
                        } else if (entry.getValue() instanceof List) {
                            // If the values in the config are a list, call the constructor of the handler with those fields.
                            HttpHandler httpHandler = (HttpHandler) ServiceUtil.constructByParameterizedConstructor(namedClass.second, (List) entry.getValue());
                            handlers.put(namedClass.first, httpHandler);
                            handlerListById.put(namedClass.first, Collections.singletonList(httpHandler));
                        }
                    }
                }
            }
            // Build add the chains to the handler list by id list.
            for (String chainName : config.getChains().keySet()) {
                List<String> chain = config.getChains().get(chainName);
                List<HttpHandler> handlerChain = new ArrayList<>();
                for (String chainItemName : chain) {
                    handlerChain.add(handlers.get(chainItemName));
                }
                handlerListById.put(chainName, handlerChain);
            }
        }
    }

    /**
     * To support multiple instances of the same class, support a naming
     * @param classLabel The label as seen in the config file.
     * @return A tuple where the first value is the name, and the second is the class.
     * @throws Exception On invalid format of label.
     */
    static Tuple<String, Class> splitClassAndName(String classLabel) throws Exception {
        String[] stringNameSplit = classLabel.split("@");
        // If i don't have a @, then no name is provided, use the class as the name.
        if (stringNameSplit.length == 1) {
            return new Tuple<>(classLabel, Class.forName(classLabel));
        } else if (stringNameSplit.length > 1) { // Found a @, use that as the name, and
            return new Tuple<>(stringNameSplit[1], Class.forName(stringNameSplit[0]));
        }
        throw new Exception("Invalid format provided for class label: "+ classLabel);
    }

    // Exposed for testing only.
    static void setConfig(String configName) throws Exception {
        config = HandlerConfig.load();
        initHandlers();
        initPaths();
    }
}

```

The following is an example of handler.yml for multiple chains configuration. 

```
# handler.yml
---
enabled: true

#------------------------------------------------------------------------------
# Support individual handler chains for each separate endpoint
#
# handlers  --  list of handlers to be used across chains in this microservice
#               including the routing handlers for ALL endpoints
#           --  format: <fully qualified handler class name>@<optional:given name>
# chains    --  allows forming of [1..N] chains, which could be wholly or 
#               used to form handler chains for each endpoint
#               ex.: default chain below, reused partially across multiple endpoints
# paths     --  list all the paths to be used for routing within the microservice
#           ----  path: the URI for the endpoint (ex.: path: '/v1/parties/search/get')
#           ----  method: the operation in use (ex.: 'post')
#           ----  exec: handlers to be executed -- this element forms the list and 
#                       the order of execution for the handlers
# 
# IMPORTANT NOTES:
# - to avoid executing a handler, it has to be removed/commented out in the chain
#   the old(er) enabled:boolean for a middleware handler will be ignored
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
  - com.networknt.swagger.SwaggerHandler@specification
  - com.networknt.security.JwtVerifyHandler@security
  - com.networknt.body.BodyHandler@body
  - com.networknt.audit.AuditHandler@audit
  - com.networknt.sanitizer.SanitizerHandler@sanitizer
  - com.networknt.validator.ValidatorHandler@validator
  # Party Domain specific cross-cutting concerns handlers
  - com.example.apifoundation.party.transactionid.TransactionIDHandler@txnID
  - com.example.apifoundation.service.audit.ServiceAuditHandler@serviceAudit
  - com.example.apifoundation.party.id.PartyIDHandler@partyID
  - com.example.api.security.fga.handler.FGAHandler@fga
  # Routing Handlers
  - com.networknt.health.HealthGetHandler@health
  - com.networknt.info.ServerInfoGetHandler@info
  - com.example.api.party.parties.handler.PartiesSearchGetPostHandler@searchHandler
  - com.example.api.party.parties.handler.PartiesRelationshipsGetPostHandler@relationshipsHandler

chains:
  default:
    - exception
    - metrics
    - traceability
    - correlation
    - specification
    # commented out - disabled in this execution chain   - security
    - body
    - audit
    - sanitizer
    - validator
    - txnID
    - serviceAudit

paths:
  - path: '/v1/parties/search/get'
    method: 'post'
    exec:
      - default
      - fga
      - searchHandler
  - path: '/v1/parties/relationships/get'
    method: 'post'
    exec:
      - default
      - partyID
      - relationshipsHandler
  - path: '/v1/health'
    method: 'get'
    exec:
      - health
  - path: '/v1/server/info'
    method: 'get'
    exec:
      - info
```


[middleware]: /concern/
