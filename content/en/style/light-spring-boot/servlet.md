---
title: "Servlet"
date: 2019-03-06T19:15:13-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Most of the existing Spring Boot applications are built on top of embedded Servlet container. By default, Tomcat is used but users can easily switch from Tomcat to Undertow Servlet container transparently. This light-4j module only works with Undertow Servlet container as it injects light-4j and light-rest-4j OpenAPI 3.0 handlers to the Undertow core http server to form a chain of handlers to handle the request and response. With this inject, the controll will go to the light-4j handlers and then pass it to the Spring Boot controllers. 

### Config

Within this module spring-servlet, there is only one class called LightConfig.java


```
package com.networknt.spring.servlet;

import com.networknt.handler.Handler;
import com.networknt.handler.OrchestrationHandler;
import io.undertow.servlet.api.DeploymentInfo;
import org.springframework.boot.web.embedded.undertow.UndertowDeploymentInfoCustomizer;
import org.springframework.boot.web.embedded.undertow.UndertowServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class LightConfig {

    @Bean
    UndertowServletWebServerFactory embeddedServletWebFactory() {
        UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
        factory.addDeploymentInfoCustomizers(new UndertowDeploymentInfoCustomizer() {
            @Override
            public void customize(DeploymentInfo deploymentInfo) {
                Handler.init();
                deploymentInfo.addInitialHandlerChainWrapper(handler -> {
                    return new OrchestrationHandler(handler);
                });
            }
        });

        return factory;
    }
}
```

The customizer method just wraps the passed in handler with light-4j OrchestrationHandler and the OrchestrationHandler will run the entire chain defined in the handler.yml in the config folder which can be externalized. The embeddedServletWebFactory is a Spring bean and the class is annotated as @Configuration. 

In order to load this LightConfig during Spring Boot application startup, the Spring Boot application need to be annotated with @ComponentScan("com.networknt")

Here is a typical application class. 

```
package com.networknt.servlet;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@ComponentScan("com.networknt")
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```


### Dependencies

This module also depending on some of the typical modules in light-4j and light-rest-4j to handle RESTful requests. With all dependencies defined in the pom.xml in this module, your Spring Boot application only need to depending on this module in order to work. 

If you want to leverage other light-4j middleware handles, you need to either add them individually to your Spring Boot application pom.xml or just open an PR to add to this module.

After you generate a Spring Boot application from https://start.spring.io/, the default will have Tomcat embedded. You need to make some change in the pom.xml to exclude Tomcat and add spring-boot-starter-undertow and light-4j spring-servlet. 

Here is an example pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.networknt</groupId>
	<artifactId>servlet</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>servlet</name>
	<description>Demo project for Light Spring Boot Servlet</description>

	<properties>
		<java.version>1.8</java.version>
		<version.light-4j>1.5.32</version.light-4j>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-undertow</artifactId>
		</dependency>
		<dependency>
			<groupId>com.networknt</groupId>
			<artifactId>spring-servlet</artifactId>
			<version>${version.light-4j}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

