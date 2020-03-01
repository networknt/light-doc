---
title: "Node Environment Variable"
date: 2020-02-25T16:57:27-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the [oauth config tutorial](/tutorial/open-banking/client/oauth-config/), we have created to client for the react-client application. One has the redirect URI https://ob.lightapi.net for production, and another has the redirect URI https://localhost:3000 for local development. There are two client_ids available, and we have to put the client_id into the react-client application login function to access the /oauth/code endpoint. 

Here is the login function in ResponsiveDrawer.js

```
    const login = () => {
        window.location = "https://obsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e75&user_type=customer&state=1222";
    }

```

The client_id in the above function is for development, and the redirect URI is https://localhost:3000 in the database script. 

When we build the react-client for production with `yarn build`, we need to find a way to replace the client_id to `f7d42348-c647-4efb-a52d-4c5787421e74` with redirect URI https://ob.lightapi.net in the database record. 


To make it possible, we need to leverage the `NODE_ENV` documented in https://create-react-app.dev/docs/adding-custom-environment-variables/

Let's change the login function to the following.

```
    const login = () => {
        // this is the production redirect with redirect uri https://ob.lightapi.net
        let url = "https://obsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e74&user_type=customer&state=1222";
        if(process.env.NODE_ENV !== 'production') {
            // this is development redirect with redirect uri https://localhost:3000
            url = "https://obsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e75&user_type=customer&state=1222";
        } 
        window.location = url;
    }
```

When you compile the app with `npm run build` or `yarn build`, the minification step will strip out the `if block`. And the resulting bundle will be smaller with only the production URL. 

Before we can enable the `yarn startHttps` and use https://localhost to develop and test the react application locally with the login-view interaction, we need to have both ob and `obsignin` folder to host the two SPA applications. Every time the react-client code is changed, we need to run `yarn build` and copy the build folder to `~/networknt/light-config-test/light-router/local-openbanking/ob` and restart the light-router instance to load the new site. 

With the https://localhost:3000 supported, we can remove the ob folder from the local-openbanking folder as we will never run the production site locally anymore.

After removing the ob folder from local-openbanking under light-router, we need to update the docker-compose only to load the obsign.lightapi.net virtal host. 

Here is the updated docker-commpose.yml

```
version: '2'

services:

  light-router:
    image:  networknt/light-router:latest
    ports:
    - 8443:8443
    extra_hosts:
    - "obsignin.lightapi.net:192.168.1.144"
    - "ob.lightapi.net:192.168.1.144"
    volumes:
    - ./config:/config
    - ./obsignin/build:/obsignin/build
    # - ./ob/build:/ob/build
    network_mode: host
```

As you can see, we have commented out the /ob/build volume mapping. We need to do the same for docker-compose-debug.yml as well.

In both config and debug folder, we need to update the virtual-host.yml to comment out the ob.lightapi.net virtual host. Here is the virtual-host.yml in the debug folder. 

```
hosts:
  - domain: obsignin.lightapi.net
    path: /
    base: /home/steve/networknt/light-config-test/light-router/local-openbanking/obsignin/build
    #base: /obsignin/build
    transferMinSize: 10245760
    directoryListingEnabled: false
  # - domain: ob.lightapi.net
  #   path: /
  #   base: /home/steve/networknt/light-config-test/light-router/local-openbanking/ob/build
  #   #base: /ob/build
  #   transferMinSize: 10245760
  #   directoryListingEnabled: false
```

When all the changes are done, we can restart the light-router instance either in Docker or in IDE to start development again. 

There is one thing that we need to be aware of. When you are using /etc/hosts to map a domain to a local IP address, you might comment it out later one to switch to the Internet DNS. The browser might have a cached site or cached DNS that needs to be cleared. 

### Chrome

Chrome caches the site and DNS for a long time, and you have to do something to clean it up. To refresh a website, you can click the refresh icon or open the developer tool and right-click the refresh icon and select `Empty cache and hard reload` to clear the cache. 

To reset the DNS cache, you need to paste address `chrome://net-internals/#dns` and click `Clear host cache`. 

### Firefox

Firefox caches the site and DNS only for a short period of time, so you just need to wait a little bit for the cache to expire. 

With all the changes above, we can use https://localhost:3000 for local development and use https://ob.lightapi.net for the test cloud. We only need to build the react-client once we need to deploy to the test cloud. 

