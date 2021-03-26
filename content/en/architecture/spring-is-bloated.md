---
title: "Spring is bloated"
date: 2017-11-06T11:14:26-05:00
description: "Spring is bloated"
categories: []
keywords: [architecture]
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

Over the years, Spring seemed to be the replacement of JEE servers with IoC
container and light weight servlet container as its foundation. Especially
recently, Spring Boot brings in an easy development model and increases 
developer productivity dramatically.

However, there are two issues or limitations in Spring applications. 

### Spring is bloated and it becomes too heavy

When Spring was out, it was only a small core with IoC contain and it was
fast and easy to use. Now, I cannot even count how many Spring Components
available today. In order to complete with JEE, Spring basically implemented all
replacements of JEE and these are heavy components.


### Most Spring applications are based on old servlet API and it is slow.

Another issue with Spring is due to the foundation of servlet container
which was designed over ten years ago without multi-core, NIO etc in
consideration. There is a little improvement in Servlet 3.1 but it wasn't
right due to backward compatible requirement. 

I did a performance test on both Spring Boot and my own Light Java Framework and found that Spring Boot is 44 times slower. The performance test code and results can be found [here](https://github.com/networknt/light-example-4j/tree/master/performance).

The test result for Spring Boot was based on the embedded tomcat server; later on, I switched to Undertow servlet container for Spring Boot. The Undertow Servlet container is faster but still over 20-30 times slower than Light Java Framework which is built on top of the Undertow core http server. 

The 20-30 times difference between the two is due to Servlet overhead and Spring
Boot overhead and it is very significant.

After I published the performance test results, one of the Spring developers pointed me to a new approach to build Spring Boot application with Netty. The performance is getting better but is still very slow compared with Light Java.

### Memory Footprint

During these tests, I observed that Spring Boot with an embedded servlet container uses at least 5 times more memory. This is a big difference in cloud computing as memory is very expensive. 

### Conclusion

Given above reasons, there is no way that Spring Boot can be used as a
light weight platform for microservices. It is too heavy and two slow. And if you
compare the codebase on both Spring Boot and Light Java, you can see Light Java
code is small and easy to understand without any annotations. 


