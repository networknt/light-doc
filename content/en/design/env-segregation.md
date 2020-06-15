---
title: "Environment Segregation"
date: 2018-02-13T20:50:25-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "design"
    weight: 50
weight: 50
aliases: []
toc: false
draft: false
reviewed: true
---


Environment segregation in APIs has primarily two goals:

* __Prevent cross fire__

In a vast majority, applications maintain segregation between test and production environments to ensure privacy, data integrity, separation of concerns, etc. To this end, an API framework needs to offer the ability to prevent access across this divide.

For example, test environments should not be able to access production services or resources, nor should production applications accidentally access test services resources such as persisted stores.

* __Support multiple different instances per environment for testing__

API frameworks need to offer the ability to support various models across an organization.

For example, in the case of separating instances of an application, separating databases, and sharing infrastructure, one instance of a service running in a test environment can connect to its own back-end resource, such as database, queues, topics, log files, etc.

Different departments will hit their own test instance, which connects to their particular database.

The above use cases are pretty complicated even in a traditional monolithic application. In a microservices architecture, it is even more complicated as there are too many services involved, and each service can have many instances.


#### To prevent cross fire

In the [light-4j](https://github.com/networknt/light-4j) framework, we are using [light-oauth2](https://github.com/networknt/light-oauth2)
to distinguish between test and production environments, or even different test phases within a larger test environment.

For each environment (or test phase), there is an OAuth2 provider to sign JWT tokens with its private key. All the services deployed to that articular environment will have the public key from the provider. In this case, even if a UAT service is trying to access a production service, the token provided by the caller cannot be verified due to an invalid signature, as a server provided the token in a different environment with different keys. Given the configuration mentioned above, there is no way we can have a crossfire if the configuration for each application and service is correctly implemented.

In turn, how can we prevent mistakes from occurring when deploying services in a cloud environment? How can we ensure that the configuration files won't be wrong during deployment?

In our ecosystem, we encourage automatic deployments without human intervention to eliminate human errors during deployment. At the same time, we employ the
[light-config-server](https://github.com/networknt/light-config-server) to serve configurations specific to each environment. These configurations are persisted in separate GitHub repositories and managed by different teams or people.


#### To support multiple different instances per environment for testing

One of our customers has an API developed by one line of business and shared by another line of business. Each line of business has its own test database, log files, configurations, with a different set of reference tables to be populated for their own testing. Furthermore, the API can connect to other resource APIs downstream and needs to access the correct
resources of those respective APIs.

To employ this model, the same API has multiple instances, each accessing its own resources, and for each of their API releases, the organization has to start several API instances connected to different databases and hand the proper instance to individual QA teams to test.

Ideally, the test environment backend should be consolidated to share the same reference data and employ similar data sets as the production environment. Unfortunately, the current model is in place for close to a decade, and it is too costly to consolidate the test data.

The solution we provided is to have an environment variable in the server.yml configuration file, to indicate which environment
(backend) database the instance is connecting. This additional attribute will be registered in Consul for service discovery, and the service discovery logic will check the variable to add additional filtering if it already exists.

The above logic also resolves service-to-service calls, binding to the same environment. For the original client, for example, Java EE based web server or test tools like SOATest or Load Runner, it must either use our client module to make the service discovery with an additional environment or is has to manually do the lookup on Consul to find the right instance with the right environment tag.

This solution will normally be used only in test environments, not production, which employs a single backend.

For added security, in production, the environment variable should not be specified in server.yml at all, eliminating a potential configuration error; therefore, all available instances for a given serviceId will be returned and load balanced. A certification process will certify the server.yml provisioned in production from the portal, and if there is an environment tag, it won't pass. This further guarantees the production environment's integrity.


#### Summary

The solution mentioned above replaces the solution built today in the existing service.

The existing solution passes an environment header in the request to indicate the testing environment of choice, while an interceptor based on aspect-oriented programming determines not only which database to connect to based on the header value, but also which log files to write messages to, which queues to access, etc.

There is no environment header specified in requests to production environments, and there should be only one database or set of resources configured by default.

By resolving cross-fire and multiple test instances at the framework level in a micro-service architecture using service discovery and the API's server.yml configuration, there is no need to pass in, or propagate, request header values while ensuring proper resource access and correct propagation in the invocation chain.
