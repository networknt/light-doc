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
reviewed: true
---

Tutorials give developers step by step guidance on how to use each component or framework. As Light is an ecosystem that delivers enterprise scale microservices, there are a lot of middleware handlers, frameworks, infrastructure services and tools that need to be covered with tutorials. In this section, some of the most important tutorials are listed and more will be added to the list.


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
     + [Skywalking](/tutorial/tracing/skywalking/)
  * [Routing](/tutorial/routing/)
     + [Multi Static Handler Chains](/tutorial/routing/handler/)
     + [Manually route requests to a handler](/tutorial/routing/manual/)
  * [Client](/tutorial/client/)
     + [Connect to public HTTPS site](/tutorial/client/public-https/)
     + [Standalone Consul Discovery](/tutorial/client/consul-discovery/)
     + [Client Dependencies](/tutorial/client/dependencies/)
     + [Client Configuration](/tutorial/client/configuration/)
     + [Client Usage](/tutorial/client/code/)
  * [Security](/tutorial/security/)     
     + It is essential to understand the [basic terminologies and formats of certs and keys][] before proceeding other tutorials.
     + How to set up your service to [bootstrap from light-oauth2 key service][] for JWT signature verification public key certificate distribution.
     + How to [encrypt and decrypt sensitive info][] from config files. 
     + How to use [customized decryptor][] to decrypt sensitive properties in config files.
     + How to use [keystore and truststore][] for one-way and two-way TLS.
     + How to create keystore for the [light-config-server TLS][] and bootstrap.truststore for light-4j service.
     + A real example on [adding server public key certificate to client.truststore][] to access public service on the Internet with HTTPS.
     + How to [install a self-signed or commercial CA-signed server certificate][] to light-4j service
     + How to get free [Let's Encrypt certificate][] for public-facing light-4j secd rvice
     + How to [convert CA-signed certificate to server.keystore][] that can be used by light-4j service
     + Running [light-4j service on port 443][] using iptables
     + How to use [environment variables][] for config values
     + How to create [self-signed client and server keystore] for light-4j server
     + How to create [self-signed JWT key and cert] for light-oauth2 and light-4j service
     + How to [generate keystore and truststore from PFX] for light-4j service
     + How to use [Okta OAuth][] 2.0 provider for light-4j service


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
     + [Light Reference](/tutorial/rest/openapi/light-reference/)
     + [Open Banking](/tutorial/open-banking/)
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

- [Light-kafka](/tutorial/light-kafka/)

- [Light-aws-lambda](/tutorial/aws-lambda/)
  * [Lambda Framework](/tutorial/aws-lambda/lambda-framework/)
  * [Lambda Proxy](/tutorial/aws-lambda/lambda-proxy/)

- [Light-chaos-monkey](/tutorial/chaos-monkey/)
  * [Petstore Chaos Monkey](/tutorial/chaos-monkey/petstore/)


- [Light-mq](/tutorial/mq/)

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
  * [React Login View](/tutorial/oauth/login-view/)
  * [Local Form Authentication](/tutorial/oauth/form-auth-local/)
  * [Test Cloud Deployment and Access](/tutorial/oauth/test-cloud/)
  * [Light-Portal Authenticator](/tutorial/oauth/light-portal-authenticator/)
  * [Update MySQL tables in Docker](/tutorial/oauth/update-mysql/)

- [OAuth-Kafka](/tutorial/oauth-kafka/)
  * [Local Dev](/tutorial/oauth-kafka/local-dev/)

- [Light-portal](/tutorial/portal/)
  * [Light Portal Local Build](/tutorial/bot/light-portal-local/)
  * [Start Portal Service](/tutorial/portal/start-portal-service/)
  * [Start Local Portal Light-Router](/tutorial/portal/local-router/)
  * [React View UI Developer](/tutorial/portal/developer/react-ui/)
  * [Portal View](/tutorial/portal/view/)
  * Portal Services
     + [user-management](/tutorial/portal/user-management/) 
     + [market-place](/tutorial/portal/market-place/)
  * Developer
     + [React View Developer][]
     + [Full Stack Developer][]
     + [Debug with JUnit Test][]
     + [Debug with Hybrid Server][]
        
- [Light-gateway](/tutorial/gateway/)
  * [Server proxy](/tutorial/gateway/server-proxy/)
  * [Client proxy server proxy](/tutorial/gateway/client-proxy-server-proxy/)
  
- [Light-proxy](/tutorial/proxy/)
  * Swagger 2.0 Backend
     + [Swagger backend service](/tutorial/proxy/swagger-backend/)
     + [Swagger Proxy](/tutorial/proxy/swagger-proxy/)
  * OpenAPI 3.0 Backend
     + [OpenAPI backend service](/tutorial/proxy/openapi-backend/)
     + [OpenAPI Proxy](/tutorial/proxy/openapi-proxy/)
  * [Proxy Docker](/tutorial/proxy/docker/)
  * [Confluent Schema Registry](/tutorial/proxy/schema-registry/)
  * [Enable Metris on Proxy Server](/tutorial/proxy/proxy-metrics/)
- [Light-router](/tutorial/router/)
  * [Router Assisted Service Discovery](/tutorial/common/discovery/router/)
  * [Deploy light-router for taiji-faucet service](/tutorial/router/taiji-faucet/)
  * [Add a react single page application to taiji-faucet router](/tutorial/router/router-spa/)
  * [Virtual hosts on taiji-faucet router](/tutorial/router/virtual-host/)
  * [Virtual hosts on light-portal](/tutorial/router/light-portal/)
- [Light-reference](/tutorial/reference/)
  * [Update Reference Table](/tutorial/reference/sql-update/)
  
- [Light-mesh](/tutorial/mesh/)
  * [KsqldbConsumer](/tutorial/mesh/kafka/ksqldb/)

- [Kafka-sidecar](/tutorial/kafka-sidecar/)
  * [Confluent Local installation on Ubuntu](/tutorial/kafka-sidecar/confluent-local/)
  * [Confluent Docker Compose](/tutorial/kafka-sidecar/confluent-docker/)
  * [Local development with Confluent Local](/tutorial/kafka-sidecar/local-dev/)
  * [Local binary format with external config](/tutorial/kafka-sidecar/local-binary/)


- [Http-sidecar](/tutorial/http-sidecar/)

- [Light-scheduler](/tutorial/scheduler/)
  * [Local Confluent][] to support local development and test.
  * [Local development][] with [Local Confluent][]
  * [API Endpoints][] with Postman and Curl command line.
  * [Three-node Cluster][] with fat jar and externalized config folders.
  * [Cluster Debug][] with docker-compose and IDE mix.

- [Light-controller](/tutorial/controller/)
  * [Petstore Registry][] to show users how to the light-controller for service interaction.
  * [Service Discovery][] to show users how to leverage the light-controller for global registry and discovery. 
  * [Kubernetes Cluster][] to show users how to use the light-controller in a Kubernetes cluster for instance management and goblal discovery. 
  * [Local IDE][] to integrate light-scheduler, light-controller and petstore API
  * [Local Docker][] to integrate light-scheduler, light-controller and petsotre API with multiple nodes docker-compose.

- [Light-config-server](/tutorial/config-server/)
  * [Mongo Provider in Debug Mode](/tutorial/config-server/mongo-debug/)
  * [View Development][] with local config server as backend.


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
  * Clone and build [dependency](/tutorial/bot/dependency/) in develop branches for Light developers
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
[customized decryptor]: /tutorial/security/customized-decryptor/
[keystore and truststore]: /tutorial/security/keystore-truststore/
[adding server public key certificate to client.truststore]: /tutorial/security/publickey-truststore/
[Install a self-signed or commercial CA-signed server certificate]: /tutorial/security/install-certificate/
[Let's Encrypt certificate]: /tutorial/security/lets-encrypt/
[convert CA-signed certificate to server.keystore]: /tutorial/security/ca-certificate/
[basic terminologies and formats of certs and keys]: /tutorial/security/term-format/
[light-4j service on port 443]: /tutorial/security/port443/
[light-config-server TLS]: /tutorial/security/config-server-cert/
[React View Developer]: /tutorial/portal/developer/react-ui/
[Full Stack Developer]: /tutorial/portal/developer/full-stack/
[Debug with JUnit Test]: /tutorial/portal/developer/debug-unit-test/
[Debug with Hybrid Server]: /tutorial/portal/developer/debug-hybrid-server/
[environment variables]: /tutorial/security/env-var-config/
[self-signed client and server keystore]: /tutorial/security/self-signed-certificate/
[self-signed JWT key and cert]: /tutorial/security/self-signed-jwt-key/
[generate keystore and truststore from PFX]: /tutorial/security/pfx-certificate/
[Local Confluent]: /tutorial/scheduler/local-confluent/
[Local development]: /tutorial/scheduler/local-dev/
[API Endpoints]: /tutorial/scheduler/api-endpoint/
[Three-node Cluster]: /tutorial/scheduler/local-cluster/
[Cluster Debug]: /tutorial/scheduler/cluster-debug/
[Petstore Registry]: /tutorial/controller/petstore-registry/
[Service Discovery]: /tutorial/controller/service-discovery/
[Kubernetes Cluster]: /tutorial/controller/kubenetes-cluster/
[Local IDE]: /tutorial/controller/local-ide/
[Local Docker]: /tutorial/controller/local-docker/
[View Development]: /tutorial/config-server/local-view/
[Okta OAuth]: /tutorial/security/okta-oauth/
