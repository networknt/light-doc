---
title: "Local Form Authentication"
date: 2019-07-19T09:07:12-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before we dive into the details, here is a video that walks through the demo and configurations.

{{< youtube T3-jOEkNuQ4 >}}

### Introduction

Recently, we have completed the form authentication in the light-oauth2 authorization code service. Now, we support Basic Authentication, Form Authentication, and SPENGO SSO. 

While working on the form authentication, I have to set up my local environment in a production-like configuration before deploying it to the cloud to provide services to our users. 

The setup consists of: 

* light-oauth2 started with a docker-compose for all eight services along with a MySQL database. 

* light-router started with another docker-compose to handle the SPA security with light-spa-4j StateLessAuthHandler and serve two virtual hosts lightapi.net (portal) and signin.lightapi.net(sign in). 


### Prepare Environment

As we are trying to mimic the production environment on the local desktop, we want to make sure that it behaves the same. With at least two virtual hosts that are served by the light-router instance, we need to update the /etc/hosts to map the DNS. Add the following line to the /etc/hosts file. 

```
192.168.1.144   lightapi.net signin.lightapi.net
```
You must change the IP address if you want to try it on your local. You can find your IP with `ifconfig` command.

When we start the router, we are using 8443 as the port number in docker-compose or start it standalone in the IDE for debugging. However, we don't want to see the port number on the browser. So we need to map the default https port 443 to 8443 on my local. Please follow this [tutorial](/tutorial/security/port443/) to set up the `iptables`. 

All the light-oauth2 services will be registered to the consul server running locally, to start it. 

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-consul.yml up -d
```

### Light-oauth2

We need to start the light-oauth2 services with one of the supported databases. In this case, we are using MySQL as the backend database in a docker-compose. 

You can find the configuration files along with a docker-compose file in light-config-test/light-oauth2/local-consul folder. 

Here is the final docker-compose. Please note that network_mode is the `host` for all services. 

```
  version: '2'
  services:
    mysqldb:
      image: mysql:5.7.16
      ports:
        - 3306:3306
      volumes:
        - ./light-oauth2/mysql:/docker-entrypoint-initdb.d
      network_mode: host    
      environment:
        MYSQL_ROOT_PASSWORD: rootpassword
        MYSQL_USER: mysqluser
        MYSQL_PASSWORD: mysqlpw
    oauth2-code:
      image: networknt/oauth2-code
      ports:
        - "6881:6881"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-code:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-code"
      #    env: "dev"
    oauth2-token:
      image: networknt/oauth2-token
      ports:
        - "6882:6882"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-token:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-token"
      #    env: "dev"
    oauth2-service:
      image: networknt/oauth2-service
      ports:
        - "6883:6883"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-service:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-service"
      #    env: "dev"
    oauth2-client:
      image: networknt/oauth2-client
      ports:
        - "6884:6884"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-client:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-client"
      #    env: "dev"
    oauth2-user:
      image: networknt/oauth2-user
      ports:
        - "6885:6885"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-user:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-user"
      #    env: "dev"
    oauth2-key:
      image: networknt/oauth2-key
      ports:
        - "6886:6886"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-key:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-key"
      #    env: "dev"
    oauth2-refresh-token:
      image: networknt/oauth2-refresh-token
      ports:
        - "6887:6887"
      volumes:
        - ./light-oauth2/mysql/config/oauth2-refresh-token:/config
      environment:
        - STATUS_HOST_IP=lightapi.net
      network_mode: host    
      depends_on:
        - mysqldb
      #logging:
      #  driver: "gelf"
      #  options:
      #    gelf-address: "udp://localhost:12201"
      #    tag: "oauth2-refresh-token"
      #    env: "dev"

```

Since all light-oauth2 services will be deployed behind the light-router, so there is no need to enable the CORS handlers for in-flight requests. We need to disable the CORS handler in the cors.yml file. Here is one of the examples. 

```
enabled: false
allowedOrigins:
- http://localhost:3000
allowedMethods:
- GET
- POST
- PUT
- DELETE
```

As we don't have the light-oauth2 console hooked up yet, we don't have a chance to create a new client_id/client_secret pair. We need to rely on the create_mysql.sql script to populate a client for our test. There is an existing bootstrap client, and we just need to modify it with the following redirect_uri. The is the URI that will handle the authorization code returned from the light-oauth2 code service. The real handler is the light-spa-4j StatelessAuthHandler which is wired in the light-router handler.yml file. 

```
'https://lightapi.net/authorization'
```

To start the light-oauth2 services. 

```
cd ~/networknt/light-config-test/light-oauth2/local-consul
docker-compose up -d
```

### Light-router

The light-oauth2 consists of eight microservices which are listening to different ports when they are started with a docker-compose. To make sure that these services can be accessed as static IP and standard HTTPS port 443, we are going to deploy a light-router instance in front of light-oauth2 instances. 

The light-oauth2 is part of the light-portal, so we don't need to create a separate configuration folder. We can reuse the light-config-test/light-router/local-portal configuration folder for the exact purpose. Of course, the folder contains configuration files and virtual hosts for other portal services and sites. 

We first need to add a brand new virtual host called `signin` for the form authentication of the light-oauth2 authorization code flow. 

The source code of this single page application can be found at light-oauth2/login-view. 

To map the new site in the docker-compose, we need to copy the build folder of the login-view to the same folder `~/networknt/light-config-test/light-router/local-portal/signin` folder. 

To build the React SPA, go to the light-oauth2/login-view folder, and run.

```
npm run build
```

We need to add a volume mapping in the docker-compose.yml for the light-router. 

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    networks:
    - localnet
    ports:
    - 8443:8443
    volumes:
    - ./config:/config
    - ./faucet/build:/faucet/build
    - ./webclient/build:/webclient/build
    - ./lightapi/build:/lightapi/build
    - ./taiji/build:/taiji/build
    - ./signin/build:/signin/build
#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true

```

Note that we have added the following line to map the ./signin/build folder into the container /signin/build folder. 

```
    - ./signin/build:/signin/build
```

As the light-router instance is dealing with both lightapi.net and signin.lightapi.net domains, we need to update the cors.yml to allow these two domains to access each other. 

```
enabled: true
allowedOrigins:
  - http://localhost:3000
  - https://signin.lightapi.net
  - https://lightapi.net
allowedMethods:
  - GET
  - POST
  - PUT
```

We also need to update the client.yml to change the default redirect_uri for the authorization code flow to `https://lightapi.net/authorization` instead of default `https://localhost:8080/authorization_code`. This URI must match with the redirect_uri defined in the MySQL database which is required by the OAuth 2.0 specification. Also, the server_url is changed from localhost to lightapi.net as the router will be running inside a docker container. 


```
  token:
    # The scope token will be renewed automatically 1 minutes before expiry
    tokenRenewBeforeExpired: 60000
    # if scope token is expired, we need short delay so that we can retry faster.
    expiredRefreshRetryDelay: 2000
    # if scope token is not expired but in renew windown, we need slow retry delay.
    earlyRefreshRetryDelay: 4000
    # token server url. The default port number for token service is 6882.
    server_url: https://lightapi.net:6882
    # token service unique id for OAuth 2.0 provider
    serviceId: com.networknt.oauth2-token-1.0.0
    # the following section defines uri and parameters for authorization code grant type
    authorization_code:
      # token endpoint for authorization code grant
      uri: "/oauth2/token"
      # client_id for authorization code grant flow. client_secret is in secret.yml
      client_id: f7d42348-c647-4efb-a52d-4c5787421e72
      # the web server uri that will receive the redirected authorization code
      redirect_uri: https://lightapi.net/authorization
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
      - petstore.r
      - petstore.w

```

In the handler.yml, we need to make the following update. 

* Add StatelessAuthHandler to the middleware handler definition. 

```
- com.networknt.auth.StatelessAuthHandler@stateless
```

* Add the above handler alias `stateless` to the default middleware handler chain. 

```
  default:
    - exception
    #- metrics
    - traceability
    - correlation
    - cors
    - stateless
    - header
    - path
    - router

```

* Add a new chain for the light-oauth2 code service that needs to skip the stateless. 

```
  code:
    - exception
    - traceability
    - correlation
    - cors
    - header
    - path
    - router
```

* Add a new path `/authorization` to accept the authorization code redirect. Basically, the request will be handled by the StatelessAuthHandler that is part of the default chain. The StatelessAuthHandler will receive the authorization code and go to the token service to get a JWT token then save the access token as well as `csrf` token in the secure cookies with HTTPS connection. 

```
  - path: '/authorization'
    method: 'GET'
    exec:
      - default
```

* Update the `/oauth2/code` from default chain to code chain to bypass the StatelessAuthHandler.

```
  # oauth2-code
  - path: '/oauth2/code'
    method: 'get'
    exec:
      - code
  - path: '/oauth2/code'
    method: 'post'
    exec:
      - code

```

* Add cors handler to the `defaultHandlers` before the virtual. It allows the redirect between the lightapi.net and signin.lightapi.net in the authorization code flow. 

```
defaultHandlers:
  - cors
  - virtual

```

As we are adding a brand new site for form authentication, we need to update the virtual-host.yml to add this new host. 

```
  - domain: signin.lightapi.net
    path: /
    #base: /home/steve/networknt/light-config-test/light-router/virtual-host/lightapi/build
    base: /signin/build
    transferMinSize: 10245760
    directoryListingEnabled: false
```

Since we have added StatelessAuthHandler to the handler chain, we need to add statelessAuth.yml to the config folder.

```
# Indicate if the StatelessAuthHandler is enabled or not
enabled: true
# Once Authorization is done, which path the UI is redirected.
redirectUri: /
# Request path for the authorization code handling.
authPath: /authorization
# Cookie domain
cookieDomain: lightapi.net
# Cookie path
cookiePath: /
# Cookie maxAge
cookieMaxAge: 600
# Login uri, redirect to it once session is expired
cookieTimeoutUri: /
# Cookie secured
cookieSecure: true
```

While working on the configuration, I need to debug the light-router instance in the IDE. So I have created a new folder called debug along with config with only one file virtual-host.yml that is different. 

In the debug folder, we are using the absolute path instead of the path within the docker container. Here is the example of signin site. 

```
  - domain: signin.lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/local-portal/signin/build
    # base: /signin/build
    transferMinSize: 10245760
    directoryListingEnabled: false
```

With the above configuration, we can start the docker-compose from the light-config-test/light-router/local-portal folder.

```
docker-compose up -d
```
### Signin SPA

In the App.js, the `handleSubmit` is a very important function. 

```
  const handleSubmit = event => {
    console.log("username = " + username + " password = " + password + " remember = " + remember);
    console.log("state = " + state + " clientId = " + clientId + " userType = " + userType + " redirectUri = " + redirectUri);
    event.preventDefault();

    let data = {
      j_username: username,
      j_password: password,
      state: state,
      client_id: clientId,
      user_type: userType,
      redirect_uri: redirectUri
    };

    const formData = Object.keys(data).map(key => encodeURIComponent(key) + '=' + encodeURIComponent(data[key])).join('&');

    console.log(formData);

    fetch("/oauth2/code", {
      method: 'POST',
      redirect: 'follow',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: formData
    })
    .then(response => {
      // HTTP redirect.
      if (response.ok && response.redirected) {
        window.location.href = response.url;
      } else {
        throw Error(response.statusText);
      }
    })
    .catch(error => console.log("error=", error));
  };

```

There are some hidden form fields that are getting from the query parameter from the redirect. When calling the fetch, note that we pass the redirect: 'follow' and the special code in the if block to redict to the response.url. 

### Portal View

In the light-portal/view, we have updated the ResponsiveDrawer.js to add the following. 


```
        window.location = "https://signin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e72&user_type=employee&redirect_uri=https://lightapi.net/authorization&state=1222";

```

As you can see, this redirect to signin.lightapi.net is hardcoded now. We also add an icon button which is calling the login function above. 

```
                    <Toolbar>
                        <IconButton
                            color="inherit"
                            aria-label="Open drawer"
                            onClick={this.handleDrawerToggle}
                            className={classes.menuButton}
                        >
                            <MenuIcon />
                        </IconButton>
                        <Typography variant="h6" color="inherit" style={{ flex: 1 }}>
                            API Portal
                        </Typography>
                        <div>
                            <IconButton color="inherit" onClick={this.login}>
                                <AccountCircle/>
                            </IconButton>
                        </div>
                    </Toolbar>

```

### Test

To test it, open a browser tab and enter the URL https://lightapi.net in the address bar. Once the single page application is loaded, click the AccountCircle button on the upper right. The browser will be redirected to the signin.lightapi.net to collect the user's credential. Please use the following user to log in. 

```
username: admin
password: 123456
```

Once the login is successful, the control will be redirected back to the lightapi.net with a JWT token in the cookie. 

### Summary

To deploy the light-oauth2 in the production-like configuration on my local computer is the first step in implementing it to the cloud to provide services to our users. 

Also, this tutorial can help our users to set up a local development environment for building applications and working on the frameworks.


