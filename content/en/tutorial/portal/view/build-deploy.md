---
title: "Build and Deploy"
date: 2019-01-03T21:39:06-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 20
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous [create-react-app][] section, we had scaffolded a React application and modified it a little bit on the App.js file. In this section, we are going to deploy it to the light-portal server so that everybody from the Internet can access it. 

### Build

The first step is to build the react application with `npm run build` command line. 

```
cd ~/networknt/light-portal/view
npm run build
```

After a few second the build will complete and a new folder `build` will be created in the view folder. Before deploying the application to the portal server, let's copy the build folder to light-config-test folder and check it into the github.com

Before copying and checking in to the [light-config-test][] repository, we need to make sure that the repository is checked out in the workspace networknt folder. Also, we need to remove the existing build folder from the destination folder lightapi.

```
rm -rf ~/networknt/light-config-test/light-router/light-portal/lightapi/*
mv ~/networknt/light-portal/view/build ~/networknt/light-config-test/light-router/light-portal/lightapi/
```

Now let's check in the light-config-test repository and deploy it to the portal server. 

```
cd ~/networknt/light-config-test
git add .
git commit -m "fixes #100 update light-portal view build for light-router"
git push origin develop
```

To deploy it to the portal server, we need to login to the portal host and check out the latest code and restart the [light-router][] instance. 

```
ssh portal
cd ~/networknt/light-config-test
git pull origin develop
cd light-router/light-portal
docker-compose down
docker-compose up -d
```

### Environment

In the above section, we have built the react application and deployed it to the portal server. You can access it from https://lightapi.net now. With the progress of the tutorials, you can observe the changes on this site. 

In this section, we are going to walk through the light-portal server configurations to show you have to serve a SPA (Single Page Application) from the light-router instance with virtual host and HTTPS supported. 

You can find the docker-compose.yml file at https://github.com/networknt/light-config-test/tree/master/light-router/light-portal, and it is used to start the [light-router][] docker container. 

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

#
# Networks
#
networks:
    localnet:
        # driver: bridge
        external: true
```

As you can see in the above volumes configuration, this light-router instance hosts four different single page applications with four different domains. 

There are all accessible from your browser. 

https://faucet.taiji.io is the Taiji Blockchain Testnet Faucet site to populate currency to your wallet. 
https://demo.taiji.io is the demo web-client for users to try out the Taiji Blockchain applications.
https://lightapi.net is the light-portal to provide cloud-based [infrastructure services][] to our users.   
https://taiji.io is the official site for Taiji Blockchain, Distributed Applications and CryptoCurrencies. 

The light-portal is a multi-tenant portal. It means some big customers might have their fully customized UI than the default https://lightapi.net. For example, a company can copy and customize the light-portal view, give us a pull request on light-config-test with the final deployment code in a folder called `example`  and a new entry in the docker-compose.yml file with some updates in the config folder. They can access their own UI from https://api.example.com after that. The backend APIs are the same, and they are all built as multi-tenant support and awareness of host from the API requests. 

This kind of customization is necessary if you want to have your own branding and stylish for the API portal site. With the customization and virtual host support, your user cannot even guess this site is a cloud service provided by Network New Technologies Inc.  

If you don't want to have your customized view application, you can still customize your portal to select certain menus/apps and change the look and feel for your organization in the https://lightapi.net site after your users log into the portal. 

To check the configurations for light-router of the light-portal server, you can browse them at https://github.com/networknt/light-config-test/tree/master/light-router/light-portal/config

To learn how to use light-router for Single Page Applications, you can visit [light-router tutorial][]

To learn how to get Let's Encrypt certificate, add it to the server.keystore and update iptalbes to route traffic from port 443 to 8443, please visit [security tutorial][]

[create-react-app]: /tutorial/portal/view/create-react-app/
[light-router]: /service/router/
[infrastructure services]: /service/
[light-router tutorial]: /tutorial/router/
[security tutorial]: /tutorial/security/
[light-config-test]: https://github.com/networknt/light-config-test
