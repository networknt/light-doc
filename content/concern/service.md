---
title: "Service"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
menu:
  docs:
    parent: "concern"
    weight: 100
weight: 100	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

### Why do we re-invent the wheel 

While building a light-weight microservices platform, we need an IoC service to bind implementations to interfaces. Given light-4j has server startup and shutdown hooks, we usually only need to inject during the server startup with constructor injection only. 

We have evaluated several IoC containers and found them to be too heavy for my use cases. Also, most of them are still using XML as configuration or annotations which eliminate the benefit of configurable IoC.

There are three different injection patterns: 

* Field injection
* Setter injection
* Constructor injection

In my opinion, both Field and Setter injections are bad, and the only way injection should be used is the instructor injection. Numeric articles are talking about the pros and cons of each pattern. 

https://kinbiko.com/java/dependency-injection-patterns/
https://advancedweb.hu/2018/03/06/dependency-injection-boundaries/


### Features of Singleton Service Factory

Given above reasons, We have built our own IoC service module which is a light-weight and fast dependency injection framework without any third party dependencies. It only supports constructor inject, and the injection is done during server startup. All the objects are saved into a map, and the key is the interface class name. It can guarantee that only one instance of implementation is available during runtime. 

The Light Platform encourages developers to build microservices with Functional Programming Style. One of the fundamental principles is immutability so that the code can be optimized to take advantage of multi-core CPUs. All singleton classes should be designed as immutable and the initialized object will be cached in the service map ready to be looked up. 

Unlike other IoC container, our service module only deals with singleton during server startup with constructor injection. It gives developers an opportunity to choose from several implementations of an interface in the service.yml config file.

For example, if you have an interface with two different implementations, you can change the externalized service.yml file on production to switch between two implementations.

### YAML configuration

The following is an example of service.yml in the test folder for this module.

Please note that the sequence of binding in the config file is crucial. If one implementation uses another injected object, that object must be defined in the configuration first. 


```
# singleton service factory configuration
singletons:
- com.networknt.service.A:
  - com.networknt.service.AImpl
- com.networknt.service.B:
  - com.networknt.service.BTestImpl
- com.networknt.service.C:
  - com.networknt.service.CImpl
- com.networknt.service.Processor:
  - com.networknt.service.ProcessorAImpl
  - com.networknt.service.ProcessorBImpl
  - com.networknt.service.ProcessorCImpl
- com.networknt.service.D1, com.networknt.service.D2:
  - com.networknt.service.DImpl
- com.networknt.service.E,com.networknt.service.F:
  - com.networknt.service.EF1Impl
  - com.networknt.service.EF2Impl
- com.networknt.service.G:
  - com.networknt.service.GImpl:
      name: Sky Walker
      age: 23
- com.networknt.service.J,com.networknt.service.K:
  - com.networknt.service.JK1Impl:
      jack: Jack1
      king: King1
  - com.networknt.service.JK2Impl:
      jack: Jack2
      king: King2
- com.networknt.service.L:
  - com.networknt.service.LImpl:
      protocol: https
      host: localhost
      port: 8080
      parameters:
        key1: value1
        key2: value2
- com.networknt.service.M:
  - com.networknt.service.MImpl:
    - java.lang.String: Steve
    - int: 2
    - int: 3
- com.networknt.service.Contact:
  - com.networknt.service.ContactImpl
- com.networknt.service.License:
  - com.networknt.service.LicenseImpl
- com.networknt.service.Info:
  - com.networknt.service.InfoImpl
- com.networknt.service.Validator<com.networknt.service.Contact>:
  - com.networknt.service.ContactValidator
- com.networknt.service.Validator<com.networknt.service.License>:
  - com.networknt.service.LicenseValidator
- com.networknt.service.Validator<com.networknt.service.Info>:
  - com.networknt.service.InfoValidator

```

### How to use the Singleton Service Factory

Here is the code that gets the singleton object from service map by passing in an interface class. 

```java
A a = SingletonServiceFactory.getBean(A.class);
```

Here is another API that you can get a list of implementations for a single interface. 

```java
J[] j = SingletonServiceFactory.getBeans(J.class);
```

Here is another API that you can get an object that implements an interface with generic types. 

```java
Validator<Info> infoValidator = SingletonServiceFactory.getBean(Validator.class, Info.class);
```

### Tutorial

The service module is one of the most important modules in the light platform and a lot of other components are relying on it. To help developers to learn it quickly, we have provided a [service tutorial][] in the tutorial section. 
  
[service tutorial]: /tutorial/common/service/

