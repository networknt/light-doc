---
title: "Requirement"
date: 2018-07-04T14:47:07-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When managing the configurations, the following things need to be considered as part of the requirement. 

* Config server instance per environment

For each environment, there should be an instance of config server with config templates and key/value pairs for the environment. The authentication and authorization would be different so that developers can access the DEV config server. Operators can access the UAT config server, and Production operators can access the PROD config server. 

The config server url will be passed to the contain through environment variable.

* Multiple config template files per service.

Each service has numeric plugins, and each plugin has its configuration file to control if it is enabled and the runtime behavior. That means for each service there would be a list of configuration files per environment like DIT, SIT, UAT, and PROD. To ease the burden to manage different config files per environment, we would create config file templates instead and replace some of the values per environment from an environmental specific config server. 

* All config server instances share the same set of config template files

For each service, how many config files are needed is known. We just need to parameterize the variables per environment to create a set of templates. These templates should be the same from an environment to an environment. All templates should be checked into the git repository to trace changes. UAT and PROD can have their own repository and it is subject to the organization's security policy. If one repository is used, there should be multiple branches per environment and only certain users have write access to each branch. The promotion flow should be from DEV to UAT to PROD with pull requests. 

* Each config server will maintain a list of key/value pairs

Users who have access to the config server can input key/value pairs to replace the variables in the config file templates. For each individual service, there are a fixed number of key/value pairs. The config server provides an API to allow users to create/update/delete and query key/value pairs with proper authorization. 

* Sensitive values need to be encrypted

Some of the key/value pairs are public info like server URL etc. Some of the key/value pairs are sensitive like database password. These values need to be encrypted in the database by an encryption key known only to the config server. If the database file is stolen, no sensitive info will be revealed. For most users, they can only retrieve the masked sensitive value. Only a privileged user can see the decrypted values. 

* Minimum config files to access config server from service

If service is bootstrapped from a config server, the config server URL will be passed in as an environment variable. Every service should have a default client.yml config file as it is provided from the light-4j client module as default. This file will contain all the commercial CA certificates to ensure the connectivity to the config server. All other config files will be downloaded from config server and unzip to the config folder specified. 

* Propagate config changes to an existing service

The config files will only be reloaded during the startup. If you restart the service, it will go to the config server to get all the latest config files as default. You can pass in an environment variable to allow the service to use the locally cached config files when restarting. 

* Support multiple git repositories for config templates

For a big organization, there might be multiple LOBs and each will have their own repository to maintain their config file templates. The config server needs to support multiple repositories to load templates and categorize them with serviceId which should be unique within the same organization. All repositories will follow the same structure and they are added to the config file in the config server.

* The final config files must be sent to the service in a secure way

The final config zip file loaded from config server contains all the sensitive info that is populated by the config server. It must be secure during the transportation with TLS. Alos, all sensitive info must be encrypted that only the service itself can decrypt it. 


* The service needs to be allowed to be restarted automatically

The first time interacting with the config server must be done from an operator; however, the service must be allowed the be restarted afterward without operator involvement. If deployed to a Kubernetes cluster, the orchestrator should be allowed to shut down and startup replicas. 

* Share key/value pairs across services

It is not necessary to define key/value pair per service all the time. Sometimes, it is OK to share some key/value pairs across services. For example, the OAuth 2.0 provider URL might be the same for all services per environment. For sensitive values, they must be defined per service so that they can be managed securely. 


* certificate management template

For each truststore, we can create a template with list of certificates so that config server can create the truststore with the template. 




