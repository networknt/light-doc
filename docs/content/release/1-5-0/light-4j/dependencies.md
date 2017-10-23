---
date: 2017-10-23T13:22:33-04:00
title: Dependencies
---

```text
[INFO] Scanning for projects...
[INFO] Inspecting build with total of 29 modules...
[INFO] Installing Nexus Staging features:
[INFO]   ... total of 29 executions of maven-deploy-plugin replaced with nexus-staging-maven-plugin
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] Parent POM
[INFO] config
[INFO] utility
[INFO] handler
[INFO] status
[INFO] client
[INFO] exception
[INFO] security
[INFO] correlation
[INFO] audit
[INFO] dump
[INFO] info
[INFO] health
[INFO] body
[INFO] header
[INFO] mask
[INFO] service
[INFO] switcher
[INFO] registry
[INFO] consul
[INFO] zookeeper
[INFO] balance
[INFO] cluster
[INFO] server
[INFO] metrics
[INFO] sanitizer
[INFO] traceability
[INFO] cors
[INFO] limit
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building Parent POM 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ light-4j ---
[INFO] com.networknt:light-4j:pom:1.5.0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building config 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ config ---
[INFO] com.networknt:config:jar:1.5.0
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] \- junit:junit:jar:4.12:test
[INFO]    \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building utility 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ utility ---
[INFO] com.networknt:utility:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building handler 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ handler ---
[INFO] com.networknt:handler:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building status 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ status ---
[INFO] com.networknt:status:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building client 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ client ---
[INFO] com.networknt:client:jar:1.5.0
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] +- commons-io:commons-io:jar:2.5:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building exception 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ exception ---
[INFO] com.networknt:exception:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building security 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ security ---
[INFO] com.networknt:security:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:compile
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] |  +- commons-io:commons-io:jar:2.5:compile
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.github.ben-manes.caffeine:caffeine:jar:2.5.6:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building correlation 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ correlation ---
[INFO] com.networknt:correlation:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building audit 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ audit ---
[INFO] com.networknt:audit:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:test
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- com.networknt:correlation:jar:1.5.0:test
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:test
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building dump 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ dump ---
[INFO] com.networknt:dump:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building info 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ info ---
[INFO] com.networknt:info:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:security:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  \- com.github.ben-manes.caffeine:caffeine:jar:2.5.6:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test (scope not updated to compile)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building health 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ health ---
[INFO] com.networknt:health:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:test
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building body 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ body ---
[INFO] com.networknt:body:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] +- org.mockito:mockito-core:jar:2.10.0:test
[INFO] |  +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO] |  +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO] |  \- org.objenesis:objenesis:jar:2.6:test
[INFO] \- com.networknt:client:jar:1.5.0:test
[INFO]    +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO]    +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO]    +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO]    +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO]    +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO]    +- commons-io:commons-io:jar:2.5:test
[INFO]    \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building header 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ header ---
[INFO] com.networknt:header:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- com.networknt:client:jar:1.5.0:test
[INFO]    +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO]    +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO]    +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO]    +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO]    +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO]    +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO]    +- commons-io:commons-io:jar:2.5:test
[INFO]    \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building mask 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ mask ---
[INFO] com.networknt:mask:jar:1.5.0
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building service 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ service ---
[INFO] com.networknt:service:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building switcher 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ switcher ---
[INFO] com.networknt:switcher:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building registry 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ registry ---
[INFO] com.networknt:registry:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:service:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building consul 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ consul ---
[INFO] com.networknt:consul:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- com.networknt:service:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:compile
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] |  +- commons-io:commons-io:jar:2.5:compile
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building zookeeper 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ zookeeper ---
[INFO] com.networknt:zookeeper:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:service:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.apache.zookeeper:zookeeper:jar:3.5.3-beta:compile
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.5; omitted for duplicate)
[INFO] |  +- commons-cli:commons-cli:jar:1.2:compile
[INFO] |  \- log4j:log4j:jar:1.2.17:compile
[INFO] +- com.101tec:zkclient:jar:0.3:compile
[INFO] |  \- (org.apache.zookeeper:zookeeper:jar:3.5.3-beta:compile - version managed from 3.3.1; omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.5; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] +- org.mockito:mockito-core:jar:2.10.0:test
[INFO] |  +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO] |  +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO] |  \- org.objenesis:objenesis:jar:2.6:test
[INFO] \- org.apache.curator:curator-test:jar:4.0.0:test
[INFO]    +- (org.apache.zookeeper:zookeeper:jar:3.5.3-beta:compile - version managed from 3.3.1; scope managed from test; omitted for duplicate)
[INFO]    \- com.google.guava:guava:jar:20.0:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building balance 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ balance ---
[INFO] com.networknt:balance:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:service:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building cluster 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ cluster ---
[INFO] com.networknt:cluster:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |  |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:service:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- com.networknt:balance:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building server 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ server ---
[INFO] com.networknt:server:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:audit:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |     \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:info:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:security:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  |  \- com.github.ben-manes.caffeine:caffeine:jar:2.5.6:compile
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:body:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:consul:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:zookeeper:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:balance:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:cluster:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:balance:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:service:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- com.networknt:client:jar:1.5.0:compile
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] |  +- commons-io:commons-io:jar:2.5:compile
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building metrics 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ metrics ---
[INFO] com.networknt:metrics:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:server:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:audit:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:info:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:security:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  |  +- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  |  |  \- com.github.ben-manes.caffeine:caffeine:jar:2.5.6:compile
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- com.networknt:body:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- com.networknt:registry:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:exception:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- com.networknt:switcher:jar:1.5.0:compile
[INFO] |  |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- com.networknt:consul:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- com.networknt:zookeeper:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:balance:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- com.networknt:cluster:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:service:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:registry:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:balance:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:service:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  \- (com.networknt:client:jar:1.5.0:compile - omitted for duplicate)
[INFO] +- com.networknt:audit:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |     \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:compile
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] |  +- commons-io:commons-io:jar:2.5:compile
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:mask:jar:1.5.0:compile
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (org.apache.commons:commons-lang3:jar:3.6:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.hdrhistogram:HdrHistogram:jar:2.1.10:compile
[INFO] +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] +- org.mockito:mockito-core:jar:2.10.0:test
[INFO] |  +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO] |  +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO] |  \- org.objenesis:objenesis:jar:2.6:test
[INFO] \- org.assertj:assertj-core:jar:3.8.0:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building sanitizer 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ sanitizer ---
[INFO] com.networknt:sanitizer:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:body:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] +- org.mockito:mockito-core:jar:2.10.0:test
[INFO] |  +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO] |  +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO] |  \- org.objenesis:objenesis:jar:2.6:test
[INFO] \- org.hamcrest:hamcrest-library:jar:1.3:test
[INFO]    \- (org.hamcrest:hamcrest-core:jar:1.3:test - omitted for duplicate)
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building traceability 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ traceability ---
[INFO] com.networknt:traceability:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building cors 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ cors ---
[INFO] com.networknt:cors:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.swagger:swagger-parser:jar:1.0.32:compile
[INFO] |  +- io.swagger:swagger-core:jar:1.5.16:compile
[INFO] |  |  +- org.apache.commons:commons-lang3:jar:3.6:compile (version managed from 3.2.1)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:jar:2.8.9:compile
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.8.9:compile - omitted for conflict with 2.9.1)
[INFO] |  |  |  \- (org.yaml:snakeyaml:jar:1.18:compile - version managed from 1.17; omitted for duplicate)
[INFO] |  |  +- io.swagger:swagger-models:jar:1.5.16:compile
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  |  \- io.swagger:swagger-annotations:jar:1.5.16:compile
[INFO] |  |  +- com.google.guava:guava:jar:20.0:compile
[INFO] |  |  \- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO] |  +- org.slf4j:slf4j-ext:jar:1.6.3:compile
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  \- ch.qos.cal10n:cal10n-api:jar:0.7.4:compile
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  \- commons-io:commons-io:jar:2.5:compile (version managed from 2.4)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:test
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.6.3; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (org.apache.commons:commons-lang3:jar:3.6:compile - version managed from 3.2.1; scope updated from test; omitted for duplicate)
[INFO] |  +- (commons-io:commons-io:jar:2.5:test - version managed from 2.4; omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.6.3; omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building limit 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ limit ---
[INFO] com.networknt:limit:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- commons-codec:commons-codec:jar:1.10:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.6:test
[INFO] |  +- commons-io:commons-io:jar:2.5:test
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - omitted for duplicate)
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.mockito:mockito-core:jar:2.10.0:test
[INFO]    +- net.bytebuddy:byte-buddy:jar:1.7.4:test
[INFO]    +- net.bytebuddy:byte-buddy-agent:jar:1.7.4:test
[INFO]    \- org.objenesis:objenesis:jar:2.6:test
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] Parent POM ......................................... SUCCESS [  0.386 s]
[INFO] config ............................................. SUCCESS [  0.135 s]
[INFO] utility ............................................ SUCCESS [  0.118 s]
[INFO] handler ............................................ SUCCESS [  0.010 s]
[INFO] status ............................................. SUCCESS [  0.017 s]
[INFO] client ............................................. SUCCESS [  0.085 s]
[INFO] exception .......................................... SUCCESS [  0.097 s]
[INFO] security ........................................... SUCCESS [  0.031 s]
[INFO] correlation ........................................ SUCCESS [  0.013 s]
[INFO] audit .............................................. SUCCESS [  0.037 s]
[INFO] dump ............................................... SUCCESS [  0.009 s]
[INFO] info ............................................... SUCCESS [  0.040 s]
[INFO] health ............................................. SUCCESS [  0.009 s]
[INFO] body ............................................... SUCCESS [  0.016 s]
[INFO] header ............................................. SUCCESS [  0.014 s]
[INFO] mask ............................................... SUCCESS [  0.008 s]
[INFO] service ............................................ SUCCESS [  0.005 s]
[INFO] switcher ........................................... SUCCESS [  0.005 s]
[INFO] registry ........................................... SUCCESS [  0.020 s]
[INFO] consul ............................................. SUCCESS [  0.023 s]
[INFO] zookeeper .......................................... SUCCESS [  0.068 s]
[INFO] balance ............................................ SUCCESS [  0.008 s]
[INFO] cluster ............................................ SUCCESS [  0.021 s]
[INFO] server ............................................. SUCCESS [  0.083 s]
[INFO] metrics ............................................ SUCCESS [  0.040 s]
[INFO] sanitizer .......................................... SUCCESS [  0.011 s]
[INFO] traceability ....................................... SUCCESS [  0.008 s]
[INFO] cors ............................................... SUCCESS [  0.095 s]
[INFO] limit .............................................. SUCCESS [  0.007 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.235 s
[INFO] Finished at: 2017-10-23T14:27:40-04:00
[INFO] Final Memory: 28M/607M
[INFO] ------------------------------------------------------------------------
```
