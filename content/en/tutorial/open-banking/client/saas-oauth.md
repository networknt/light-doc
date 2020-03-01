---
title: "Remote SaaS OAuth 2.0 Provider"
date: 2020-02-25T21:02:51-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous [remote service](/tutorial/open-banking/client/remote-service/) tutorial, we have demonstrated how to leverage remote test cloud services from the local Nodejs server for UI developers. The tutorial is only suitable for pure UI developers who only know the API of the backend services and focus on the React UI application 100 percent. In most organizations, there might be some full-stack developers who have to work on both SPA and backend APIs at the same time. They need an OAuth 2.0 SaaS provider for authentication and authorization remotely so that they can focus on their business locally. 

For us, the light-oauth2 is a high-performance OAuth 2.0 provider with seven microservices, and we have planned to deploy it to our test cloud to help organizations who are not keen to install the light-oauth2 internally for development. In the long term, we also plan to offer the light-oauth2 for production service as SaaS. 

Some light-oauth2 services like the user, client, and service are deployed behind the light-portal on the test cloud, and these services can be accessed through https://lightapi.net portal once authenticated and authorized to register new clients, new services and new users. 

There are two parts offered from the light-oauth2: Seven microservices and a login-view Single Page Application. Some of the services can be accessed from the light-portal site to register clients, services, and users. Other services like code, token, and key can be accessed at dev.oauth.lightapi.net for testing. The login-view application should be deployed along with the customer client application on the light-router instance so that users can customize it according to the style of the organization. For every client application, you need a light-router instance any way for virtual hosts and reverse proxy anyway. 

Now, let's create a new configuration folder under local-openbanking and call it saas. This means we are going to leverage the SaaS OAuth2 provider remotely at https://dev.oauth.lightapi.net on the test cloud.

We still need to run the open-banking services locally with the Consul registry. Let's start the local consul server first.

### Environment

We are going to name the login-view application as login.lightapi.net virtual host, and we need to map the login.lightapi.net to the local IP address in the /etc/hosts

```
192.168.1.144   login.lightapi.net
```

If you try the tutorial on your local computer, please change the IP address to your desktop IP address.

### Consul

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

### Open Banking Services

```
cd ~/open-banking/light-config-test/local-consul
docker-compose up -d
```

You can check the services on the Consul UI with the following address. 

```
http://localhost:8500
```

### light-router

In the client.yml, we are using serviceId for token service for Consul lookup. Now, we need to comment out the serviceId and enable the server_url: https://dev.oauth.lightapi.net as below. This will allow the light-router to route the request /oauth2/code to the https://dev.oauth.lightapi.net directly without doing the local Consul lookup based on the serviceId. 

```
  token:
    cache:
      #capacity of caching TOKENs
      capacity: 200
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: https://dev.oauth.lightapi.net
    # token service unique id for OAuth 2.0 provider
    # serviceId: com.networknt.oauth2-token-1.0.0
    # set to true if the oauth2 provider supports HTTP/2
    enableHttp2: true

```

In the pathPrefixService.yml, we still need the oauth2 endpoints in the mapping, but they will be overwritten by the service_url in the HTTP headers.

```
# Whether to enable PathPrefixServiceHandler
enabled: true

# mapping from request path prefixes to serviceIds
mapping:
  # Light OAuth 2.0 APIs
  /oauth2/service: com.networknt.oauth2-service-1.0.0
  /oauth2/user: com.networknt.oauth2-user-1.0.0
  /oauth2/password: com.networknt.oauth2-user-1.0.0
  /oauth2/client: com.networknt.oauth2-client-1.0.0
  /oauth2/token: com.networknt.oauth2-token-1.0.0
  /oauth2/code: com.networknt.oauth2-code-1.0.0
  /oauth2/key: com.networknt.oauth2-key-1.0.0
  # Open Banking APIs
  /accounts: com.networknt.ob.accounts-3.1.2
  /balances: com.networknt.ob.balances-3.1.2
  /beneficiaries: com.networknt.ob.beneficiaries-3.1.2
  /direct-debits: com.networknt.ob.directDebits-3.1.2
  /offers: com.networknt.ob.offers-3.1.2
  /parties: com.networknt.ob.parties-3.1.2
  /products: com.networknt.ob.products-3.1.2
  /scheduled-payments: com.networknt.ob.scheduledPayments-3.1.2
  /standing-orders: com.networknt.ob.standingOrders-3.1.2
  /statements: com.networknt.ob.statements-3.1.2
  /transactions: com.networknt.ob.transactions-3.1.2

```

In the statelessAuth.yml, we need to update the redirect, deny and cookie domain to localhost.

```
---
# Indicate if the StatelessAuthHandler is enabled or not
enabled: true
# Once Authorization is done, which path the UI is redirected.
redirectUri: https://localhost:3000
# An optional redirect uri if the user deny or cancel the authorization on the Consent page. Default to redirectUri if missing.
denyUri: https://localhost:3000
# Request path for the authorization code handling.
authPath: /authorization
# Request path for the deny authorization handling to remove HttpOnly access-token and other cookies
logoutPath: /logout
# Cookie domain
cookieDomain: localhost
# Cookie path
cookiePath: /
# Cookie maxAge
cookieMaxAge: 600
# Login uri, redirect to it once session is expired
cookieTimeoutUri: /
# Cookie secured
cookieSecure: true

```

As the https://localhost:3000 is served by the Nodejs server, only need the virtual host login.lightapi.net for the light-oauth2 login-view. Below is an example that we start the light-router as a standalone app or in an IDE for debugging. If you want to start the light-router in Docker, please change the base to /login/build. 

```
hosts:
  - domain: login.lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/local-openbanking/login/build
    #base: /login/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  # - domain: ob.lightapi.net
  #   path: /
  #   #base: /home/steve/networknt/light-config-test/light-router/local-openbanking/ob/build
  #   base: /ob/build
  #   transferMinSize: 10245760
  #   directoryListingEnabled: false

```

Since we don't have light-oauth2 services running locally, we need to rely on https://dev.oauth.lightapi.net to access these APIs. In the light-router, we support service_url HTTP header to specify the static URL of the service for static routing. 

We first need to add dev.oauth.lightapi.net into the whitelist in the router.yml

```
---
# Client Router Configuration
# As this router is built to support discovery and security for light-4j services,
# the outbound connection is always HTTP 2.0 with TLS enabled.

# If HTTP 2.0 protocol will be accepted from incoming request.
http2Enabled: true

# If TLS is enabled when accepting incoming request. Should be true on test and prod.
httpsEnabled: true

# Max request time in milliseconds before timeout to the server.
maxRequestTime: 300000

# Rewrite Host Header with the target host and port and write X_FORWARDED_HOST with original host
rewriteHostHeader: true

# Reuse XForwarded for the target XForwarded header
reuseXForwarded: false

# Max Connection Retries
maxConnectionRetries: 3

# Whitelist of hosts for static routing. Use Regex to do wildcard match
hostWhitelist:
  - dev.oauth.lightapi.net
```

The consul is running in the local, and we need to update the consul.yml to update the consulUrl with login.lightapi.net as it is mapped in the /etc/hosts to my IP address. 

```
# Consul URL for accessing APIs
consulUrl: http://login.lightapi.net:8500
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

We need to add dev.oauth.lightapi.net to the cors.yml

```
enabled: true
allowedOrigins:
  - https://login.lightapi.net
  - https://ob.lightapi.net
  - https://localhost:3000
  - https://dev.oauth.lightapi.net
allowedMethods:
  - GET
  - POST
  - PUT

```


### react-client

We need to update the setupProxy.js with target https://login.lightapi.net which is hosted in the local light-router instance. Unlike other configuration, we need to redirect to localhost, and the Nodejs server must support a specific origin https://login.lightapi.net with credentials as true. For information on how to configure the cors module, please check https://www.npmjs.com/package/cors#configuration-options

```
module.exports = function(app) {
  var corsOptions = {
    origin: 'https://login.lightapi.net',
    credentials: true,
    optionsSuccessStatus: 200 // some legacy browsers (IE11, various SmartTVs) choke on 204
  }
  app.use(cors(corsOptions));

  app.use(proxy('/accounts', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/balances', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/beneficiaries', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/direct-debits', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/offers', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/parties', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/products', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/scheduled-payments', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/standing-orders', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/statements', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/transactions', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/authorization', { target: 'https://login.lightapi.net', secure: false }));
  app.use(proxy('/logout', { target: 'https://login.lightapi.net', secure: false }));
};

```

As you can see, we basically redirect all the API access to the https://login.lightapi.net

In the ResponsiveDrawer.js file, we need to use the https://localhost:3000 redirect URL in the dev mode. 

```
    const login = () => {
        // this is the production redirect with redirect uri https://ob.lightapi.net
        let url = "https://login.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e74&user_type=customer&state=1222";
        if(process.env.NODE_ENV !== 'production') {
            // this is development redirect with redirect uri https://localhost:3000
            url = "https://login.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e75&user_type=customer&state=1222";
        } 
        window.location = url;
    }
```

For each component that is accessing the open-banking endpoint, we need to update it with the path only for the URL and comment out the long-lived token. 

```
  const url = '/products';
  const headers = {
    //Authorization: 'Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ',
    'x-fapi-financial-id': 'OB'
  };

```

### login-view

We also need to update the login-view to pass the service_url in the request header for the login page. When building the login-view from the light-oauth2 repository, we need to use the following command. 


```
yarn build-dev
```

And then copy the build folder to the light-config-test. 

```
cd ~/networknt/light-config-test/light-router/local-openbanking/login
rm -rf build
cp -r ~/networknt/light-oauth2/login-view/build .
```

### Test

Now, you can restart the light-router instance and then start the Nodejs server.

```
yarn startHttps
```

Although we have written so many tutorials about the light-oauth2, it is still very hard for users to manage it locally. If you have Internet access, it is very easy to use our dev.oauth.lightapi.net for authentication and authorization. All you need to do is to register the user and client on the portal server at https://lightapi.net

Please be aware that the dev.oauth.lightapi.net will be reset very often for security. If you client_id and user_id doesn't work anymore, please register a new one again. Or you can send us a PR to add them at https://github.com/networknt/light-config-test/blob/master/light-oauth2/test-cloud/light-oauth2/mysql/create_mysql.sql


