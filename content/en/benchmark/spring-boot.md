---
title: "Spring Boot"
date: 2019-03-04T08:26:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Recently, one of the big customers asked if we can integrate light-4j middleware handlers to address cross-cutting concerns to Spring Boot applications. They want to leverage the light-4j middleware handlers as they are faster, smaller and easier to customize. Most importantly, these handlers work with Light infrastructure to form an ecosystem to support microservices architecture. In this way, they can migrate their thousands of Spring Boot applications to Light without changing the existing controllers and business logic. The different development teams welcome the idea as they can still use Spring Boot to develop or maintain their applications without knowing the light-4j middleware injections as the DevOps team handles it in the pipeline. 


The customer realizes the [performance difference] between Light-4j and Spring Boot as they have compared both with their typical applications. While we are working on the integration in [light-spring-boot][] project, we can easily inject Light-4j handlers to the Spring Boot application in the same process. So we did a simple performance test to figure out what is the difference between the Light-4j implementation vs. Spring Boot controller. 

If you want to watch a video walk through. 

{{< youtube JcVJr00cDOw >}}

### Setup

We first generated a Spring Boot application with https://start.spring.io/ with `Web` dependency with Spring Boot 2.1.3 version. Once the project is downloaded to the local, we switch the default Tomcat embedded server to Undertow

We have updated the pom.xml to exclude the Tomcat server and added Undertow to the dependency. 


```
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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

```

### Spring Controller

We have added a RestController as following. This is a very simple controller that returns `OK` to the `/spring-boot` path. 

```
package com.networknt.demo;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {
    @RequestMapping("/spring-boot")
    public String greeting() {
        return "OK";
    }
}

```


### Light-4j Handler

Now let's inject a light-4j handler to return `OK` to the `/light-4j` path in the SpringBootApplication. 

```
package com.networknt.demo;

import io.undertow.Handlers;
import io.undertow.Undertow;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.server.RoutingHandler;
import io.undertow.servlet.api.DeploymentInfo;
import io.undertow.util.Headers;
import io.undertow.util.Methods;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.embedded.undertow.UndertowBuilderCustomizer;
import org.springframework.boot.web.embedded.undertow.UndertowDeploymentInfoCustomizer;
import org.springframework.boot.web.embedded.undertow.UndertowServletWebServerFactory;
import org.springframework.context.annotation.Bean;

import java.util.Collections;

@SpringBootApplication
public class DemoApplication {

	@Bean
	UndertowServletWebServerFactory embeddedServletWebFactory() {
		UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
		factory.addDeploymentInfoCustomizers(new UndertowDeploymentInfoCustomizer() {
			@Override
			public void customize(DeploymentInfo deploymentInfo) {
				deploymentInfo.addInitialHandlerChainWrapper(handler -> {
							return Handlers.path()
									.addPrefixPath("/", handler)
									.addPrefixPath("/light-4j", getTestHandler());
						}
				);
			}
		});

		return factory;
	}

	static HttpHandler getTestHandler() {
		return new HttpHandler() {
			@Override
			public void handleRequest(HttpServerExchange httpServerExchange) throws Exception {
				httpServerExchange.getResponseSender().send("OK");
			}
		};
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}

```

As you can see, we have added a customizer to handle the `/light-4j` with an Undertow HttpHandler, and Spring Boot still handles other paths. Light-4j handler is the same with externalized configuration and other enterprise-level features. When you are using light-spring-boot, this injection will be done transparently with the pom.xml dependency and Spring @component scanning. 

### Start the Server

To build the server, run the following maven command.

```
mvn clean install
```

To start the server, run the following command.

```
java -jar target/performance-0.0.1-SNAPSHOT.jar
```

Now you can test the two paths by issue the following curl commands. The response will be the same `OK`. 

Spring Boot

```
curl http://localhost:8080/spring-boot
```

Light-4j

```
curl http://localhost:8080/light-4j
```

### Performance

We are using the wrk to test the perforamnce and a wrk pipeline.lua to generate as many requests as possible. To learn how to use wrk on Linux and Mac, please visit https://doc.networknt.com/tool/wrk-perf/. If you don't want to create the pipeline.lua file manually, then you can checkout the [microservices-framework-benchmark][] which has the file in the root folder. 


Now, let's test the performance of these two different paths. 

Spring Boot

```
steve@freedom:~/networknt/microservices-framework-benchmark$ wrk -t8 -c128 -d30s http://localhost:8080 -s pipeline.lua --latency -- /spring-boot 64
Running 30s test @ http://localhost:8080
  8 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    19.89ms   13.15ms 103.76ms   64.24%
    Req/Sec    30.50k     2.27k   37.76k    69.58%
  Latency Distribution
     50%   18.79ms
     75%   29.92ms
     90%   41.53ms
     99%    0.00us
  7286784 requests in 30.02s, 0.96GB read
Requests/sec: 242744.25
Transfer/sec:     32.64MB
```

Light-4j

```
steve@freedom:~/networknt/microservices-framework-benchmark$ wrk -t8 -c128 -d30s http://localhost:8080 -s pipeline.lua --latency -- /light-4j 64
Running 30s test @ http://localhost:8080
  8 threads and 128 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.93ms  588.47us  15.83ms   62.78%
    Req/Sec   617.40k    49.68k    1.24M    88.89%
  Latency Distribution
     50%    0.98ms
     75%    1.54ms
     90%    2.23ms
     99%    0.00us
  147690816 requests in 30.10s, 13.89GB read
Requests/sec: 4906794.22
Transfer/sec:    472.63MB
```

In comparing the two results, we can see that the light-4j path achieves about 20 times the throughput of spring-boot path and the latency is 0.93ms vs. 19.89ms. Imagine if you wanted to reach the same throughput and same latency of light-4j path: how many Spring Boot instances would be needed? We have one customer who switched from Spring Boot to Light-4j for their service that collects data from IoT devices and they reduced from 250 medium size AWS VMs to only 4 small size AWS VMs with much better performance. 

The test was done on my desktop computer with AMD 2700X and 32GB memory. The [source code][] can be found at Github and you can run it locally. 

[microservices-framework-benchmark]: https://github.com/networknt/microservices-framework-benchmark
[performance difference]: https://github.com/networknt/microservices-framework-benchmark
[light-spring-boot]: https://github.com/networknt/light-spring-boot
[source code]: https://github.com/networknt/light-example-4j/tree/develop/springboot/performance
