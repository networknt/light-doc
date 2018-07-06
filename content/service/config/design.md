---
title: "Config Server Design"
date: 2018-07-04T14:47:23-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

There will be a lot of sensitive info in the config files such as database password, private key store password etc. The security design is very crucial for the config server and the interaction during the service startup. 

As the sensitive info in the config file is encrypted, we need to pass the encryption key to the service as an environmental variable in the deployment file. The same encryption key must be previously set up on the config server for the service from the same operator so that the config server will use it to encrypt the sensitive info. 

To allow the service to start automatically, the encryption key must be put into the deployment config file for the Kubernetes cluster. And it will be revealed to someone who has access to the deployment script. To prevent an attacker to steal the encryption key to decrypt the sensitive info from the config files manually, there are two extra variables that need to be built into the service itself. 

* The encryption algorithm

There will be numeric encryption algorithm supported by both config server and light-4j services. Each service will pick up one to be built in and only the development team knows it. When the service is initialized on the config server, the product owner of the service will provide picks up the right algorithm for the service. 

* Salt

Each service will have a random number built in as salt. The salt will be applied to the encryption key in order to work. The service owner will also need to provide the salt to the config server during service initialization. 

For deployment operators, they cannot see the algorithm and salt for the service they are deploying. So the encryption key is not enough to do any damage. For the product owner, they are not able to access the encryption key. For a big organization, there is a separate team that is responsible for different tasks. For a small organization, to limit the information spread to more people, you can turn off the salt for the test environment so that secret will only be revealed to a limited number of people. 



