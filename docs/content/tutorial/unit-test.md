---
date: 2016-10-12T17:05:47-04:00
title: Unit Test
---

## Introduction

For framework developers, writing unit tests is a must. These tests will help developers
to verify if their changes in the code base break anything. Without unit tests, it would
be so scary to make code change. For API/service developers, it is encouraged to write
unit tests to facilitate CI/CD devops flow.

There are two type of test cases and they serve different purposes and will be triggered
at different step of life cycle. In this section, we are focus on unit tests. There is
another article that discusses [integration test](integration-test/).

## Unit Test

Unit tests will be executed every time the code is changed and they maybe executed many
times in a day. They need to run as fast as possible so that developers likes to run
these tests. They need to cover the majority of branches in the code but there is no
dependencies to external systems.

The unit tests sit in /src/test folder as default and they have class name like *Test and
method names prefix with test* are test cases.

## JUnit 5

Majority of our unit tests were written in Junit 4.12 and we are in a transition to Junit
5 now. To use Junit5, you need to put the dependency into the parent pom.

```
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-engine</artifactId>
                <version>${version.junit}</version>
                <scope>test</scope>
            </dependency>

```

The current junit version is 5.0.1

## Maven surefire plugin

Unit test cases are executed by surefire plugin and this is defined in the pom.xml in
the parent pom.

```
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <dependencies>
                    <dependency>
                        <groupId>org.junit.platform</groupId>
                        <artifactId>junit-platform-surefire-provider</artifactId>
                        <version>1.0.1</version>
                        <scope>compile</scope>
                    </dependency>
                </dependencies>
            </plugin>

```

## How to run junit test without triggering integration test during development.

If you only want junit tests to be executed, please user package.

```
mvn clean package
```


If you want to run both unit tests and integration tests.

```
mvn clean verify
```
