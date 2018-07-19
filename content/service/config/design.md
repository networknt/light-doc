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

### Security Interaction

There will be a lot of sensitive info in the config files such as database password, private key store password etc. The security design is very crucial for the config server and the interaction during the service startup. 

As the sensitive info in the config file is encrypted, we need to pass the encryption key to the service as an environmental variable in the deployment file. The same encryption key must be previously set up on the config server for the service from the same operator so that the config server will use it to encrypt the sensitive info. 

To allow the service to start automatically, the encryption key must be put into the deployment config file for the Kubernetes cluster. And it will be revealed to someone who has access to the deployment script. To prevent an attacker to steal the encryption key to decrypt the sensitive info from the config files manually, there are two extra variables that need to be built into the service itself. 

* The encryption algorithm

There will be numeric encryption algorithms supported by both config server and light-4j services. Each service will pick up one to be built in and only the development team knows it. When the service is initialized on the config server, the product owner of the service will provide the right algorithm for the service. 

* Salt

Each service will have a random number built in as salt. The salt will be applied to the encryption key in order to work. The service owner will also need to provide the salt to the config server during service initialization. 

For deployment operators, they cannot see the algorithm and salt for the service they are deploying. So the encryption key is not enough to do any damage. For the product owner, they are not able to access the encryption key. For a big organization, there is a separate team that is responsible for different tasks. For a small organization, to limit the information spread to more people, you can turn off the salt for the test environment so that secret will only be revealed to a limited number of people. 

### Database Encryption

To protect the interaction between the config server and services is not enough. We need to encrypt the database values so that even the database file is stolen, no sensitive info will be leaked. This requires that an encryption key must be available for the config server and it must be saved in the server instance instead in the database itself. We cannot generate random key as this key is needed to recover database if the current instance is down and we have to start another instance. To maximize the security, the key is splitted into two parts for two operators. Both of them will input their key to recover the real encryption key for the database values. 

### Framework

There are a lot interaction with the UI and only our team is accessing the API of config server. It is a lot easily to utilize light-hybrid-4j with react-schema-form for the UI. The other benefit is that it can be integrated seamlessly with light-portal in our deployment. But for customers who doesn't want to use light-portal, they can still use the light-config-server independently. 

### Database

The light-portal is using ArangoDB by default. As we are dealing with key/value most of the case, it would be natural to use ArangoDB; however, we need to provide another plugin layer to access SQL database in case someone wants to use it independently. Once the database layer is abstracted, it would be very easy to replace it with service.yml configuration change. 

### Model and Schema

The latest model-schema can be found at https://github.com/networknt/model-config/tree/develop/hybrid/config-server





