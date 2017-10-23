---
date: 2017-10-23T13:22:33-04:00
title: Dependencies
---

```text
[INFO] Scanning for projects...
[INFO] Inspecting build with total of 4 modules...
[INFO] Installing Nexus Staging features:
[INFO]   ... total of 4 executions of maven-deploy-plugin replaced with nexus-staging-maven-plugin
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] Parent POM
[INFO] swagger-meta
[INFO] swagger-security
[INFO] swagger-validator
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building Parent POM 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ light-rest-4j ---
[INFO] com.networknt:light-rest-4j:pom:1.5.0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building swagger-meta 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ swagger-meta ---
[INFO] com.networknt:swagger-meta:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
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
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:audit:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |     \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- io.swagger:swagger-parser:jar:1.0.32:compile
[INFO] |  +- io.swagger:swagger-core:jar:1.5.16:compile
[INFO] |  |  +- org.apache.commons:commons-lang3:jar:3.6:compile
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:jar:2.8.9:compile
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.8.9:compile - omitted for conflict with 2.9.1)
[INFO] |  |  |  \- (org.yaml:snakeyaml:jar:1.17:compile - omitted for conflict with 1.18)
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
[INFO] |  \- commons-io:commons-io:jar:2.4:compile
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.6.3; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (org.apache.commons:commons-lang3:jar:3.6:compile - scope updated from test; omitted for duplicate)
[INFO] |  +- (commons-io:commons-io:jar:2.5:test - omitted for conflict with 2.4)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:test - omitted for duplicate)
[INFO] +- ch.qos.logback:logback-classic:jar:1.2.3:test
[INFO] |  +- ch.qos.logback:logback-core:jar:1.2.3:test
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.6.3; omitted for duplicate)
[INFO] \- junit:junit:jar:4.12:test
[INFO]    \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building swagger-security 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ swagger-security ---
[INFO] com.networknt:swagger-security:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:security:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
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
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:status:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] +- com.networknt:swagger-meta:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:audit:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- io.swagger:swagger-parser:jar:1.0.32:compile
[INFO] |  |  +- io.swagger:swagger-core:jar:1.5.16:compile
[INFO] |  |  |  +- (org.apache.commons:commons-lang3:jar:3.2.1:compile - omitted for conflict with 3.6)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  |  +- com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:jar:2.8.9:compile
[INFO] |  |  |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.8.9:compile - omitted for conflict with 2.9.1)
[INFO] |  |  |  |  \- (org.yaml:snakeyaml:jar:1.17:compile - omitted for conflict with 1.18)
[INFO] |  |  |  +- io.swagger:swagger-models:jar:1.5.16:compile
[INFO] |  |  |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  |  |  \- io.swagger:swagger-annotations:jar:1.5.16:compile
[INFO] |  |  |  +- com.google.guava:guava:jar:20.0:compile
[INFO] |  |  |  \- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO] |  |  +- org.slf4j:slf4j-ext:jar:1.6.3:compile
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  |  \- ch.qos.cal10n:cal10n-api:jar:0.7.4:compile
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  \- (commons-io:commons-io:jar:2.4:compile - omitted for conflict with 2.5)
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
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test (scope not updated to compile)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - version managed from 2.8.9; omitted for duplicate)
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
[INFO] \- org.apache.commons:commons-text:jar:1.1:test
[INFO]    \- (org.apache.commons:commons-lang3:jar:3.5:test - omitted for conflict with 3.6)
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building swagger-validator 1.5.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ swagger-validator ---
[INFO] com.networknt:swagger-validator:jar:1.5.0
[INFO] +- com.networknt:config:jar:1.5.0:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.7; omitted for duplicate)
[INFO] |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.9.1:compile
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  \- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- org.yaml:snakeyaml:jar:1.18:compile
[INFO] +- com.networknt:swagger-meta:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:status:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  \- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- com.networknt:audit:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  +- io.swagger:swagger-parser:jar:1.0.32:compile
[INFO] |  |  +- io.swagger:swagger-core:jar:1.5.16:compile
[INFO] |  |  |  +- (org.apache.commons:commons-lang3:jar:3.2.1:compile - omitted for conflict with 3.5)
[INFO] |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  |  +- com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:jar:2.8.9:compile
[INFO] |  |  |  |  +- (com.fasterxml.jackson.core:jackson-core:jar:2.8.9:compile - omitted for conflict with 2.9.1)
[INFO] |  |  |  |  \- (org.yaml:snakeyaml:jar:1.17:compile - omitted for conflict with 1.18)
[INFO] |  |  |  +- io.swagger:swagger-models:jar:1.5.16:compile
[INFO] |  |  |  |  +- (com.fasterxml.jackson.core:jackson-annotations:jar:2.8.9:compile - omitted for conflict with 2.9.0)
[INFO] |  |  |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.22; omitted for duplicate)
[INFO] |  |  |  |  \- io.swagger:swagger-annotations:jar:1.5.16:compile
[INFO] |  |  |  +- com.google.guava:guava:jar:20.0:compile
[INFO] |  |  |  \- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO] |  |  +- (org.slf4j:slf4j-ext:jar:1.6.3:compile - omitted for conflict with 1.7.25)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  \- commons-io:commons-io:jar:2.5:compile
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:swagger-security:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:utility:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
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
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  +- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] |  |  \- com.github.ben-manes.caffeine:caffeine:jar:2.5.6:compile
[INFO] |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:swagger-meta:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.6.3; omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (org.bitbucket.b_c:jose4j:jar:0.6.1:compile - omitted for duplicate)
[INFO] +- com.networknt:body:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- com.networknt:exception:jar:1.5.0:compile
[INFO] |  |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:handler:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.networknt:status:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- commons-codec:commons-codec:jar:1.10:compile
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- com.networknt:handler:jar:1.5.0:compile
[INFO] |  +- (com.networknt:config:jar:1.5.0:compile - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.9; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:compile - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:compile - omitted for duplicate)
[INFO] |  \- (io.undertow:undertow-core:jar:1.4.20.Final:compile - omitted for duplicate)
[INFO] +- io.undertow:undertow-core:jar:1.4.20.Final:compile
[INFO] |  +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO] |  +- org.jboss.xnio:xnio-api:jar:3.3.8.Final:compile
[INFO] |  \- org.jboss.xnio:xnio-nio:jar:3.3.8.Final:runtime
[INFO] |     \- (org.jboss.xnio:xnio-api:jar:3.3.8.Final:runtime - omitted for duplicate)
[INFO] +- com.networknt:json-schema-validator:jar:0.1.10:compile
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile - version managed from 2.8.7; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- org.slf4j:slf4j-ext:jar:1.7.25:compile
[INFO] |  |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] |  \- org.apache.commons:commons-lang3:jar:3.5:compile
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.9.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.9.1:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] +- org.owasp.encoder:encoder:jar:1.2.1:compile
[INFO] +- org.bitbucket.b_c:jose4j:jar:0.6.1:compile
[INFO] |  \- (org.slf4j:slf4j-api:jar:1.7.25:compile - version managed from 1.7.21; omitted for duplicate)
[INFO] +- com.networknt:client:jar:1.5.0:test (scope not updated to compile)
[INFO] |  +- (com.networknt:utility:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:config:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.networknt:status:jar:1.5.0:test - omitted for duplicate)
[INFO] |  +- (com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:test - version managed from 2.8.7; omitted for duplicate)
[INFO] |  +- (org.slf4j:slf4j-api:jar:1.7.25:test - version managed from 1.7.21; omitted for duplicate)
[INFO] |  +- (commons-codec:commons-codec:jar:1.10:test - omitted for duplicate)
[INFO] |  +- (org.owasp.encoder:encoder:jar:1.2.1:test - omitted for duplicate)
[INFO] |  +- (org.apache.commons:commons-lang3:jar:3.6:test - omitted for conflict with 3.5)
[INFO] |  +- (commons-io:commons-io:jar:2.5:compile - scope updated from test; omitted for duplicate)
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
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] Parent POM ......................................... SUCCESS [  0.364 s]
[INFO] swagger-meta ....................................... SUCCESS [  0.462 s]
[INFO] swagger-security ................................... SUCCESS [  0.086 s]
[INFO] swagger-validator .................................. SUCCESS [  0.061 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.678 s
[INFO] Finished at: 2017-10-23T15:35:08-04:00
[INFO] Final Memory: 23M/607M
[INFO] ------------------------------------------------------------------------

```