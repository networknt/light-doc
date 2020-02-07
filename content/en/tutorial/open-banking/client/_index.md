---
title: "Open Banking SPA Client"
date: 2019-12-14T21:14:26-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [router][] step, we have exposed the services to the Internet through https://ob.lightapi.net subdomain. Eventually, the services will be consumed by other services, single page applications or mobile applications. In this step, we are going to build a consumer application using React and deploy it to the light-router instance on the portal as another virtual host. 

In the previous steps, we are using long-lived tokens to access services. With a single page application, we can also close the entire loop of OAuth 2.0 authorization code flow. 

To start a React application, we need to create a repo on the github under open-banking organization. Then clone the repo to the open-banking folder under home directory. 

```
git clone git@github.com:open-banking/react-client.git
npx create-react-app react-client
yarn start
```

A browser tab will be started automatically after the node server is up with the SPA. Let's shutdown the server and check in the generated code. 

```
git add .
git commit -m "initial checkin"
git push origin master
```


In order to test the client locally, we are going to setup the consul, light-oauth2 and light-router locally following this [local portal][] tutorial until the step to start the light-router. 

To make the ob.lightapi.net subdomain works locally, we need to update the /etc/hosts file to add this subdomain name. After the modification, we have the following line in the /etc/hosts file. 

```
192.168.1.144   lightapi.net signin.lightapi.net ob.lightapi.net
```

If you try the tutorial on your local, please change the IP address to your local address. 


The default database script for the light-oauth2 only contains admin user and a client that is for the portal. We need to add two more users and a client with redirect set up as https://ob.lightapi.net

The dbscript is located at https://github.com/networknt/light-config-test/tree/master/light-oauth2/local-consul/light-oauth2/mysql

Add the following two users.

```
-- add open banking users
INSERT INTO user_profile(user_id, user_type, first_name, last_name, email, roles, password)
VALUES('stevehu', 'customer', 'Steve', 'Hu', 'stevehu@gmail.com', 'user', '1000:5b39342c202d37372c203132302c202d3132302c2034372c2032332c2034352c202d34342c202d31362c2034372c202d35392c202d35362c2039302c202d352c202d38322c202d32385d:949e6fcf9c4bb8a3d6a8c141a3a9182a572fb95fe8ccdc93b54ba53df8ef2e930f7b0348590df0d53f242ccceeae03aef6d273a34638b49c559ada110ec06992');
INSERT INTO user_profile(user_id, user_type, first_name, last_name, email, roles, password)
VALUES('ericbroda', 'customer', 'Eric', 'Broda', 'ericbroda@gmail.com', 'user', '1000:5b39342c202d37372c203132302c202d3132302c2034372c2032332c2034352c202d34342c202d31362c2034372c202d35392c202d35362c2039302c202d352c202d38322c202d32385d:949e6fcf9c4bb8a3d6a8c141a3a9182a572fb95fe8ccdc93b54ba53df8ef2e930f7b0348590df0d53f242ccceeae03aef6d273a34638b49c559ada110ec06992');

```

Add the following Open Banking Client.

```
-- Open Banking SPA demo client registration to https://ob.lightapi.net/authorization
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e74', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'browser', 'Open Banking SPA Demo', 'Open Banking Single Page Application React Client', 'accounts', '{"c1": "361", "c2": "67"}', 'https://ob.lightapi.net/authorization', 'admin');
-- Open Banking SPA demo client registration to https://localhost:3000/authorization
INSERT INTO client (client_id, client_secret, client_type, client_profile, client_name, client_desc, scope, custom_claim, redirect_uri, owner_id)
VALUES('f7d42348-c647-4efb-a52d-4c5787421e74', '1000:5b37332c202d36362c202d36392c203131362c203132362c2036322c2037382c20342c202d37382c202d3131352c202d35332c202d34352c202d342c202d3132322c203130322c2033325d:29ad1fe88d66584c4d279a6f58277858298dbf9270ffc0de4317a4d38ba4b41f35f122e0825c466f2fa14d91e17ba82b1a2f2a37877a2830fae2973076d93cc2', 'public', 'browser', 'Open Banking SPA Demo', 'Open Banking Single Page Application React Client', 'accounts', '{"c1": "361", "c2": "67"}', 'https://localhost:3000/authorization', 'admin');

```



[router]: /tutorial/open-banking/router/
[local portal]: /tutorial/portal/local-router/
