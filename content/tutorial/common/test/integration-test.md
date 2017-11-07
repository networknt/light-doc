---
title: "Integration Test"
date: 2017-11-06T17:45:44-05:00
description: "Integration Test"
categories: []
keywords: [tutorial]
menu:
  docs:
    parent: "tutorial"
    weight: 30
weight: 30
aliases: []
toc: false
draft: false
---

Not all tests are running fast and some of the tests are very complex with external
dependencies. It will take more time to run all tests so it is better to separate
unit tests and integration tests.

In this case, in the normal package, only unit tests will be executed so that can be
done really fast.

On a daily basis, we might run nightly build to include both unit tests and integration
tests together.

## Integration tests

Normally, integration test will require there are some servers running like database,
webserver etc. Or some test cases need to be repeated too many time. These tests will
be located in src/integration folder and all test classes will end with "IT".

As integration is not the default test source folder or resources folder, we need to add
this in pom.xml

```
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>1.9.1</version>
                <executions>
                    <execution>
                        <id>add-test-source</id>
                        <goals>
                            <goal>add-test-source</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>${project.basedir}\src\integration\java</source>
                            </sources>
                        </configuration>
                    </execution>
                    <execution>
                        <id>add-test-resource</id>
                        <goals>
                            <goal>add-test-resource</goal>
                        </goals>
                        <configuration>
                            <resources>
                                <resource>
                                    <!-- Don't forget <directory> label -->
                                    <directory>${project.basedir}\src\integration\resources</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

```

## Maven failsafe plugin

The integration tests will be called from maven failsafe plugin.

```
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
                <executions>
                    <execution>
                        <id>integration-test</id>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>


```


## How to run integration test.

```
mvn clean install

or

mvn clean verify
```

