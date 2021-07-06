---
title: "Light Portal Authenticator"
date: 2020-03-24T17:38:26-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

One of the essential features of light-oauth2 is its extensibility. When a bank is using an OAuth 2.0 server to authenticate users, it needs to integrate with several user management databases or services. For example, employees will be authenticated with Active Directory via LDAP or SPENGO single sign-on. For retail customers, they will be authenticated with a customer profile database in a mainframe. 

To make the integration with back end authenticators easier, we have provided the authhub module in light-oauth2. In this tutorial, we are going to walk through the process to add Light Portal Authenticator to the light-oauth2. 

Light-portal uses user-command and user-query to manage users with event sourcing and CQRS. The user-query service exposes a `loginUser` endpoint to accept an email and a password for user authentication. 

### Customized Authenticator

To allow the light-portal application to leverage the light-portal user services to authenticate users, we need to implement a new authenticator in the authhub. 

We first create a dummy interface called LightPortalAuth. 

```
package com.networknt.oauth.auth;

public interface LightPortalAuth {
}

```

And then we create the implementation class. 

```
package com.networknt.oauth.auth;

import com.networknt.client.Http2Client;
import com.networknt.cluster.Cluster;
import com.networknt.config.JsonMapper;
import com.networknt.oauth.security.LightPasswordCredential;
import com.networknt.server.Server;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.UndertowOptions;
import io.undertow.client.ClientConnection;
import io.undertow.client.ClientRequest;
import io.undertow.client.ClientResponse;
import io.undertow.security.idm.Account;
import io.undertow.security.idm.Credential;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xnio.OptionMap;

import java.net.URI;
import java.net.URLEncoder;
import java.security.Principal;
import java.util.Arrays;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicReference;

/**
 * This is the light-portal authentication class. It should be defined in the authenticate_class
 * column in client table when register the light-portal application so that this class will be
 * invoked when a user is login from light-portal site lightapi.net.
 * <p>
 * The current implementation for light-portal will use portal user-query to do authentication and
 * roles in the user profile for authorization.
 *
 * Unlike other corporate applications, there is only one type of user for the light-portal.
 *
 * The assumption is that the light-oauth2 and light-portal are deployed to the same cloud and register
 * to the same Consul cluster. If that is not the case, then you cannot use service lookup but use direct
 * url like lightapi.net/portal/query to access the portal query service.
 *
 * @author Steve Hu
 */
public class LightPortalAuthenticator extends AuthenticatorBase<LightPortalAuth> {
    private static final Logger logger = LoggerFactory.getLogger(LightPortalAuthenticator.class);
    private static final String cmd = "{\"host\":\"lightapi.net\",\"service\":\"user\",\"action\":\"loginUser\",\"version\":\"0.1.0\",\"data\":{\"email\":\"%s\",\"password\":\"%s\"}}";
    private static final String queryServiceId = "com.networknt.portal.hybrid.query-1.0.0";
    private static String tag = Server.getServerConfig().getEnvironment();
    // Get the singleton Http2Client instance
    static Http2Client client = Http2Client.getInstance();
    static ClientConnection connection;
    static Cluster cluster;

    public LightPortalAuthenticator() {
        // Get the singleton Cluster instance
        cluster = SingletonServiceFactory.getBean(Cluster.class);
        String host = cluster.serviceToUrl("https", queryServiceId, tag, null);
        try {
            connection = client.connect(new URI(host), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
        } catch (Exception e) {
            logger.error("Exception:", e);
        }
    }

    @Override
    public Account authenticate(String id, Credential credential) {
        LightPasswordCredential passwordCredential = (LightPasswordCredential) credential;
        char[] password = passwordCredential.getPassword();
        // user-query service authentication and authorization
        try {
            if(connection == null || !connection.isOpen()) {
                // The connection is close or not created.
                String host = cluster.serviceToUrl("https", queryServiceId, tag, null);
                connection = client.connect(new URI(host), Http2Client.WORKER, Http2Client.SSL, Http2Client.BUFFER_POOL, OptionMap.create(UndertowOptions.ENABLE_HTTP2, true)).get();
            }
            final CountDownLatch latch = new CountDownLatch(1);
            final AtomicReference<ClientResponse> reference = new AtomicReference<>();
            final String s = String.format(cmd, id, new String(password));
            String message = "/portal/query?cmd=" + URLEncoder.encode(s, "UTF-8");
            final ClientRequest request = new ClientRequest().setMethod(Methods.GET).setPath(message);
            request.getRequestHeaders().put(Headers.HOST, "localhost");
            connection.sendRequest(request, client.createClientCallback(reference, latch));
            latch.await();
            int statusCode = reference.get().getResponseCode();
            String body = reference.get().getAttachment(Http2Client.RESPONSE_BODY);
            if(statusCode == 200) {
                Map<String, Object> map = JsonMapper.string2Map(body);
                // {"roles":"user","id":"stevehu@gmail.com"}
                String roles = (String)map.get("roles");
                Account account = new Account() {
                    private Set<String> roles = splitRoles((String)map.get("roles"));
                    private final Principal principal = () -> id;
                    @Override
                    public Principal getPrincipal() {
                        return principal;
                    }

                    @Override
                    public Set<String> getRoles() {
                        return roles;
                    }
                };
                return account;
            }
        } catch (Exception e) {
            logger.error("Exception:", e);
            return null;
        }
        return null;
    }

    public Set<String> splitRoles(String roles) {
        Set<String> set = new HashSet<>();
        if(roles != null) {
            String[] splited = roles.split("\\s+");
            set = new HashSet<>(Arrays.asList(splited));
        }
        return set;
    }
}

```

The constructor creates the cluster and connection and caches them. In the authenticate method, it does the lookup only if necessary. The assumption is that we have light-oauth2 and light-portal service deployed on the same cloud. 


### Configuration

To make the above invocation to the light-portal query side works, we need to modify the configuration files. 

First, we need to add a consul.yml for Consul cluster connectivity. Below is the one works for the local consul running in a Docker container. 


```
# Consul URL for accessing APIs
consulUrl: http://192.168.1.144:8500
# access token to the consul server
consulToken: the_one_ring
# number of requests before reset the shared connection.
maxReqPerConn: 1000000
# deregister the service after the amount of time after health check failed.
deregisterAfter: 2m
# health check interval for TCP or HTTP check. Or it will be the TTL for TTL check. Every 10 seconds,
# TCP or HTTP check request will be sent. Or if there is no heart beat request from service after 10 seconds,
# then mark the service is critical.
checkInterval: 10s
# One of the following health check approach will be selected. Two passive (TCP and HTTP) and one active (TTL)
# enable health check TCP. Ping the IP/port to ensure that the service is up. This should be used for most of
# the services with simple dependencies. If the port is open on the address, it indicates that the service is up.
tcpCheck: false
# enable health check HTTP. A http get request will be sent to the service to ensure that 200 response status is
# coming back. This is suitable for service that depending on database or other infrastructure services. You should
# implement a customized health check handler that checks dependencies. i.e. if db is down, return status 400.
httpCheck: false
# enable health check TTL. When this is enabled, Consul won't actively check your service to ensure it is healthy,
# but your service will call check endpoint with heart beat to indicate it is alive. This requires that the service
# is built on top of light-4j and the above options are not available. For example, your service is behind NAT.
ttlCheck: true
# endpoints that support blocking will also honor a wait parameter specifying a maximum duration for the blocking request.
# This is limited to 10 minutes.This value can be specified in the form of "10s" or "5m" (i.e., 10 seconds or 5 minutes,
# respectively).
wait: 600s
# enable HTTP/2
# must disable when using HTTP with Consul (mostly using local Consul agent), Consul only supports HTTP/1.1 when not using TLS
# optional to enable when using HTTPS with Consul, it will have better performance
enableHttp2: false

```

We need to update the service.yml file to add the following section for Consul connection and registration for the LightPortalAuthenticator. 

```
- com.networknt.registry.URL:
  - com.networknt.registry.URLImpl:
      protocol: light
      host: localhost
      port: 8080
      path: consul
      parameters:
        registryRetryPeriod: '30000'
- com.networknt.consul.client.ConsulClient:
  - com.networknt.consul.client.ConsulClientImpl
- com.networknt.registry.Registry:
  - com.networknt.consul.ConsulRegistry
- com.networknt.balance.LoadBalance:
  - com.networknt.balance.RoundRobinLoadBalance
- com.networknt.cluster.Cluster:
  - com.networknt.cluster.LightCluster
# Authenticator implementation mapping
- com.networknt.oauth.auth.Authenticator<com.networknt.oauth.auth.LightPortalAuth>:
  - com.networknt.oauth.auth.LightPortalAuthenticator

```

### Test

It is very easy to create JUnit test cases to ensure that the authenticator works. Here is a test class we have created in the test folder. 


```
package com.networknt.oauth.auth;

import com.networknt.oauth.security.LightPasswordCredential;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.security.idm.Account;
import org.junit.Assert;
import org.junit.Test;

import java.util.Set;

/**
 * These are all live test case and they are depending on the light-portal hybrid-query running locally
 * and registered on a local Consul server running in Docker container. In most of the cases, these
 * test cases will be disabled. They are created for developers to debug the implementation.
 * 
 * @author Steve Hu
 */
public class LightPortalAuthenticatorTest {
    //@Test
    public void testSplitRoles() {
        String s = "user admin lightapi.net";
        LightPortalAuthenticator auth = new LightPortalAuthenticator();
        Set<String> set = auth.splitRoles(s);
        Assert.assertEquals(set.size(), 3);
        Assert.assertTrue(set.contains("user"));
    }

    /**
     * Manually inject the authenticator and test with a constructed LightPasswordCredential.
     */
    //@Test
    public void testAuthenticate() {
        Class clazz = DefaultAuth.class;
        try {
            clazz = Class.forName("com.networknt.oauth.auth.LightPortalAuth");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        Authenticator authenticator = SingletonServiceFactory.getBean(Authenticator.class, clazz);
        Assert.assertTrue(authenticator != null);
        Account account = authenticator.authenticate("stevehu@gmail.com", new LightPasswordCredential("123456".toCharArray(), null, null));
        Assert.assertTrue(account != null);
    }
}

```

The test case takes the same approach as the light-oauth2 service. It injects the authenticator from the service.yml with a given identifier interface class. Then it invokes the authenticate method with a manually constructed Credential. 


### Deployment

To deploy the new authenticator, we just need to build Docker images for the light-oauth2 services and restart the docker-compose in the local and test cloud. We need to make sure that the above configuration changes are applied to the code service. 

We also need to update the oauth2 database to set up the authenticator class in the client table for the light-portal application. 

Here are the insert statements for the local-consul and test-cloud of the light-oauth2 database script. 

```
-- light-portal client registration redirect to https://lightapi.net/authorization
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id, authenticate_class)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e72', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'mobile', 'Light Portal Web Application', 'Light Portal React Single Page Application', 'portal.r portal.w', null, 'https://lightapi.net/authorization', 'stevehu@gmail.com', 'com.networknt.oauth.auth.LightPortalAuth');
-- light-portal client registration redirect to https://localhost:3000/authorization for local testing
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id, authenticate_class)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e73', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'mobile', 'Light Portal Web Application', 'Light Portal React Single Page Application', 'portal.r portal.w', null, 'https://localhost:3000/authorization', 'stevehu@gmail.com', 'com.networknt.oauth.auth.LightPortalAuth');

```

The last column authenticate_class and the corresponding value `com.networknt.oauth.auth.LightPortalAuth` will make sure that the light-portal authenticator will be invoked for this particular client. 

In the light-config-test/light-oauth2 folder, we need to update the configuration for code service.yml in consul-local and test-cloud.

Here is the [commit](https://github.com/networknt/light-config-test/commit/b4292f7039852ff0146b38473a33c5532cdee672) for the update. 

### Demo

To be written.
