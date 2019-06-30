---
title: "Tutorial Overview"
date: 2017-11-06T17:45:44-05:00
description: "Tutorials"
categories: []
keywords: [tutorial]
menu:
  docs:
    parent: "tutorial"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Tutorials give developers step by step guidance on how to use each component or framework. As light
platform is an ecosystem to deliver enterprise scale microservices, there are a lot of middleware
handlers, frameworks, infrastructure services and tools need to be covered with tutorials. In this
section, some of the most important tutorials are listed here and more will be added into the list. 


### Cross-Cutting Concerns

- [Cross-Cutting Concerns](/tutorial/middleware/)
  * [CORS - Cross-Origin Resource Sharing](/tutorial/middleware/cors/)
  * [Prometheus for metrics](/tutorial/middleware/prometheus/)
  * [Encrypt and Decrypt](/tutorial/security/encrypt-decrypt/)
  * [Singleton Service Injection](/tutorial/common/service/)
     + [Introduction](/tutorial/common/service/introduction/)
     + [Single Interface Single Implementation](/tutorial/common/service/singlesingle/)
     + [Single Interface Multiple Implementations](/tutorial/common/service/singlemultiple/)
     + [Single Interface with Generic Type](/tutorial/common/service/generic-type/)
     + [Multiple Interfaces Single Implementation](/tutorial/common/service/multiple-single/)
     + [Multiple Interfaces Single Implementation](/tutorial/common/service/multiple-multiple/)
     + [Implementation with constructor parameters](/tutorial/common/service/constructor-parameters/)
     + [POJO Java Bean](/tutorial/common/service/java-bean/)
     + [Inject Manually for debugging](/tutorial/common/service/manual-injection/)
     + [Summary](/tutorial/common/service/summary/)
  * [Service Registry and Discovery](/tutorial/common/discovery/)
     + [Introduction and code generation](/tutorial/common/discovery/generated/)
     + [Static Configuration](/tutorial/common/discovery/static/)
     + [Dynamic service discovery with direct registry](/tutorial/common/discovery/dynamic/)
     + [Multiple API D Instances](/tutorial/common/discovery/multiple/)
     + [Consul service registry and discovery](/tutorial/common/discovery/consul/)
     + [Multiple instances with tags per environment](/tutorial/common/discovery/tag/)
     + [Access consul with acl_token for security](/tutorial/common/discovery/token/)
     + [Service discovery for Docker container](/tutorial/common/discovery/docker/)
     + [Service discovery for Kubernetes](/tutorial/common/discovery/kubernetes/)
     + [Service discovery for OpenShift](/tutorial/common/discovery/openshift/)
     + [Router Assisted Service Discovery](/tutorial/common/discovery/router/)
     + [Consul Production with TLS](/tutorial/common/discovery/consul-tls/)
     + [External Config](/tutorial/common/discovery/external-config/)
     + [Consul HTTP Health Check](/tutorial/common/discovery/http-health/)
     + [Docker-compose and Consul Production](/tutorial/common/discovery/compose-consul/)
     + [OpenTracing with Jaeger](/tutorial/tracing/jaeger/service-discovery/)
  * [Tracing](/tutorial/tracing/)
     + [Jaeger OpenTracing](/tutorial/tracing/jaeger/)
  * [Routing](/tutorial/routing/)
     + [Multi Static Handler Chains](/tutorial/routing/handler/)
  * [Client](/tutorial/client/)
     + [Connect to public HTTPS site](/tutorial/client/public-https/)
     + [Standalone Consul Discovery](/tutorial/client/consul-discovery/)
     + [Client Dependencies](/tutorial/client/dependencies/)
     + [Client Configuration](/tutorial/client/configuration/)
     + [Client Usage](/tutorial/client/code/)
  * [Security](/tutorial/security/)     
     + It is essential to understand the [basic terminologies and formats of certs and keys][] before proceeding other tutorials.
     + How to set up your service to [bootstrap from light-oauth2 key service][] for JWT signature verification public key certificate distribution.
     + How to [encrypt and decrypt sensitive info][] from secret.yml or other config files. 
     + How to use [keystore and truststore][] for one-way and two-way TLS.
     + A real example on [adding server public key certificate to client.truststore][] to access public service on the Internet with HTTPS.
     + How to [install a self-signed or commercial CA-signed server certificate][] to light-4j service
     + How to get free [Let's Encrypt certificate][] for public-facing light-4j secd rvice
     + How to [convert CA-signed certificate to server.keystore][] that can be used by light-4j service
     + Running [light-4j service on port 443][] using iptables

### API Styles

- [Light-rest-4j](/tutorial/rest/)
  * [Swagger 2.0](/tutorial/rest/swagger/)
     + [Swagger 2.0 Petstore](/tutorial/rest/swagger/petstore/)
     + [Microservices Chain Pattern](/tutorial/rest/swagger/ms-chain/)
  * [OpenAPI 3.0](/tutorial/rest/openapi/)
     + [OpenAPI 3.0 Petstore](/tutorial/rest/openapi/petstore/)
     + [Restful Database Access](/tutorial/rest/openapi/database/)
     + [Microservices Aggregate Pattern](/tutorial/rest/openapi/ms-aggregate/)
     + [Microservices Branch Pattern](/tutorial/rest/openapi/ms-branch/)
     + [Parameter Serialization](/tutorial/rest/openapi/parameter-serialization/)
     + [ServiceMesher](/tutorial/rest/openapi/servicemesher/)
  * [Spring Boot](/tutorial/springboot/)   
     + [Spring Boot Servlet](/tutorial/springboot/servlet/)
  * [Kotlin](/tutorial/kotlin/)
     + [Kotlin OpenAPI](/tutorial/kotlin/openapi/)

- [Light-graphql-4j](/tutorial/graphql/)
  * [Hello World](/tutorial/graphql/helloworld/)
  * [Star Wars](/tutorial/graphql/starwars/)
  * [Mutation](/tutorial/graphql/mutation/)
  * [Mutation IDL](/tutorial/graphql/mutation-idl/)
  * [Relay Todo](/tutorial/graphql/relay-todo/)
  * [Subscription](/tutorial/graphql/subscription/)
- [Light-hybrid-4j](/tutorial/hybrid/)
  * [Merge two services with one server](/tutorial/hybrid/merge-services/)
  * [Hello World Hybrid Server and Services](/tutorial/hybrid/hello-world/)
  * [Eventuate Todo-list Hybrid Implementation](/tutorial/hybrid/todo-list/)
  * [Portal Hybrid Service initialization](/tutorial/hybrid/hybrid-service-initial/)
  * [Portal Hybrid Query with Light Codegen Web](/tutorial/hybrid/codegen-web-portal/)
  * [Portal Hybrid Query with API Certification](/tutorial/hybrid/certification-portal/)
  * [Portal Hybrid Command with Host Menu](/tutorial/hybrid/host-menu-command-portal/)
  * [Portal Hybrid Query with Host Menu](/tutorial/hybrid/host-menu-query-portal/)
  * [Portal Hybrid Command with Schema Form](/tutorial/hybrid/schema-form-command-portal/)
  * [Portal Hybrid Query with Schema Form](/tutorial/hybrid/schema-form-query-portal/)
  * [Portal Hybrid Service with User Management](/tutorial/hybrid/user-management-portal/)
  * [Taiji Blockchain Web Client Server SPA with handler.yml](/tutorial/hybrid/web-client-spa/)
  * [Register individual handlers to Consul for discovery](/tutorial/hybrid/register-handler/)

- [Light-eventuate-4j](/tutorial/eventuate/)
  * [Getting Started](/tutorial/eventuate/getting-started/)
  * [Todo-list Tutorial](/tutorial/eventuate/todo-list/)
     + [Prepare Envionment](/tutorial/eventuate/todo-list/prepare/)    
     + [Create Project](/tutorial/eventuate/todo-list/project/)    
     + [Rest Command Side](/tutorial/eventuate/todo-list/rest-command/)    
     + [Rest Query Side](/tutorial/eventuate/todo-list/rest-query/)    
     + [Rest Test](/tutorial/eventuate/todo-list/rest-test/)    
     + [Hybrid Command Side](/tutorial/eventuate/todo-list/hybrid-command/)    
     + [Hybrid Query Side](/tutorial/eventuate/todo-list/hybrid-query/)    
     + [Hybrid Test](/tutorial/eventuate/todo-list/hybrid-test/)    
     + [Rest Docker](/tutorial/eventuate/todo-list/rest-docker/)    
     + [Hybrid Docker](/tutorial/eventuate-4j/todol-list/hybrid-docker/)    
  * [Account Management Tutorial](/tutorial/eventuate/account-management/)  
  * [Kafka Tutorial][] helps to work with Kafka in Docker container.
  * [Test Tutorial][] helps users to write unit tests, integration tests, and end-to-end tests.
  * [Developer's Tutorial][] guidelines for developers who are working on the framework or services
  * [Docker Tutorial][] helps users to work with Docker and Eventuate

- [Light-tram-4j](/tutorial/tram/)
  * [Todo list tutorial](/tutorial/tram/todo-list/)    
  * [Development API with light-tram-4j](/tutorial/tram/develop/)    

### Infrastructure Service

- [Light-oauth2](/tutorial/oauth/)
  * [How to generate long lived access token](/tutorial/oauth/longlive/)
  * [How to start services](/tutorial/oauth/start/)
  * [Authorization Code](/tutorial/oauth/code/)
  * [Token Endpoint](/tutorial/oauth/token/)
  * [Service Registration](/tutorial/oauth/service/)
  * [Client Registration](/tutorial/oauth/client/)
  * [User Management](/tutorial/oauth/user/)
  * [Key Distribution](/tutorial/oauth/key/)
  * [Client Authenticated User](/tutorial/oauth/custom/)
  * [Provider Service](/tutorial/oauth/provider/)
  * [Deploy to Kubernetes Cluster](/tutorial/oauth/kubernetes/)
  * [Deploy to Openshift Cluster](/tutorial/oauth/openshift/)
  * [Integrate Kerberos/SPNEGO on Mac](/tutorial/oauth/kerberos/)

- [Light-portal](/tutorial/portal/)
  * [Light Portal Local Build](/tutorial/bot/light-portal-local/)
  * [Start Portal Service](/tutorial/portal/start-portal-service/)
  * [Start Local Portal Light-Router](/tutorial/portal/local-router/)
  * [Portal View](/consumer/local-first/)
  * Portal Services
     + [user-management](/tutorial/portal/user-management/) 
- [Light-proxy](/tutorial/proxy/)
  * Swagger 2.0 Backend
     + [Swagger backend service](/tutorial/proxy/swagger-backend/)
     + [Swagger Proxy](/tutorial/proxy/swagger-proxy/)
  * OpenAPI 3.0 Backend
     + [OpenAPI backend service](/tutorial/proxy/openapi-backend/)
     + [OpenAPI Proxy](/tutorial/proxy/openapi-proxy/)
  * [Proxy Docker](/tutorial/proxy/docker/)
  * [Confluent Schema Registry](/tutorial/proxy/schema-registry/)
- [Light-router](/tutorial/router/)
  * [Router Assisted Service Discovery](/tutorial/common/discovery/router/)
  * [Deploy light-router for taiji-faucet service](/tutorial/router/taiji-faucet/)
  * [Add a react single page application to taiji-faucet router](/tutorial/router/router-spa/)
  * [Virtual hosts on taiji-faucet router](/tutorial/router/virtual-host/)
  * [Virtual hosts on light-portal](/tutorial/router/light-portal/)

### Tool Chain

- [Light-codegen](/tutorial/generator/)
  * [Generate Restful API with Swagger 2.0 Specification](/tutorial/generator/swagger/)
  * [Generate Restful API with OpenAPI 3.0 Specification](/tutorial/generator/openapi/)
  * [Generate Restful API in Kotlin with OpenAPI 3.0 Specification](/tutorial/generator/openapikotlin/)
  * [Generate GraphQL API with GraphQL IDL](/tutorial/generator/graphql/)
  * [Generate Hybrid Service with Schema](/tutorial/generator/hybridservice/)
  * [Generate Hybrid Server Hosts Hybrid services](/tutorial/generator/hybridserver/)
  * [Generate CQRS Eventuate based Restful API with OpenAPI 3.0 Specification](/tutorial/generator/eventuate/)
  * [Generate Swagger 2.0 Mock API with Examples](/tutorial/generator/swagger-mock/)
  * [Generate OpenAPI 3.0 Mock API with Examples](/tutorial/generator/openapi-mock/)
  * [Integrate Code generation with maven build process with Examples](/tutorial/generator/codegen-maven/)
  * [Start codegen-web to support multiple versions](/tutorial/generator/multi-version-web/)

- [Light-bot](/tutorial/bot/)
  * Clone and [build light-bot](/tutorial/bot/build-light-bot/) project and command line tool with Gradle
  * Clone and build [dependency](/tutorial/bot/dependency/) in develop branches for light platform developers
  * Set up [develop](/tutorial/bot/local-develop/) branch build and test on your local computer
  * Set up [devops](/tutorial/bot/devops-develop/) sever for develop branch build automatically
  * Build [light-portal local](/tutorial/bot/light-portal-local/) develop branch using cloud eventuate services
  * How to use [regex-replace](/tutorial/bot/regex-replace/) to replace repeated string/text across all repositories
- [Development](/consumer/light-router/)
  * [Test](/tutorial/common/test/)
     + [Unit Test](/tutorial/common/test/unit-test/)
     + [Integration test](/tutorial/common/test/integration-test/)
     + [Consumer driven contract test](/tutorial/common/test/consumer-driven-contract/)
     + [End-to-end test](/tutorial/common/test/end-to-end-test/)
  * [Debug](/tutorial/common/debug/)
     + [Debug service with IntelliJ Idea](/tutorial/common/debug/idea/) 
     + [Debug service with Eclipse](/tutorial/common/debug/eclipse/) 
- [Deployment](/consumer/light-router/)
  * [Running Light-4J Application as Linux Service](/deployment/linux_service/)
  * [Running Light-4J Application as Windows Service](/deployment/windows_service/)
  * [Install Eventuate on Windows](/deployment/windows-eventuate/)

- [Test](/tutorial/common/test/)
  * [Unit Test](/tutorial/common/test/unit-test/)
  * [Integration Test](/tutorial/common/test/integration-test/)
  * [End-to-End Test](/tutorial/common/test/end-to-end-test/)
  * [Consumer Driven Contract](/tutorial/common/test/consumer-driven-contract/)

- [Debug](/tutorial/common/debug/)
  * [IntelliJ IDEA](/tutorial/common/debug/idea/)
  * [Eclipse](/tutorial/common/debug/eclipse/)
  * [Remote Debug with Docker Container](/tutorial/common/debug/docker-remote/)

[Kafka Tutorial]: /tutorial/eventuate/kafka/
[Test Tutorial]: /tutorial/eventuate/test/
[Developer's Tutorial]: /tutorial/eventuate/developer/
[Docker Tutorial]: /tutorial/eventuate/docker/
[bootstrap from light-oauth2 key service]: /tutorial/security/bootstrap-from-key-service/
[encrypt and decrypt sensitive info]: /tutorial/security/encrypt-decrypt/
[keystore and truststore]: /tutorial/security/keystore-truststore/
[adding server public key certificate to client.truststore]: /tutorial/security/publickey-truststore/
[Install a self-signed or commercial CA-signed server certificate]: /tutorial/security/install-certificate/
[Let's Encrypt certificate]: /tutorial/security/lets-encrypt/
[convert CA-signed certificate to server.keystore]: /tutorial/security/ca-certificate/
[basic terminologies and formats of certs and keys]: /tutorial/security/term-format/
[light-4j service on port 443]: /tutorial/security/port443/

