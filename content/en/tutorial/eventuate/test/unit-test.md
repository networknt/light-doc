---
title: "Unit Test"
date: 2018-01-06T20:31:06-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Before reading specific information regarding the Unit Test, please read the general [unit test tutorial][]
This section will only focus on the specific information regarding to light-eventuate-4j. There are a lot
of modules in light-eventuate-4j and each module has its own JUnit test cases. They are all located in
each project's src/test folder. There might also be a config folder under src/test/resources/ to control
how each involved light-4j component behaves. 

## JUnit Test

Run the JUnit test cases in the each component modules src/test folder. You can run individual tests within
IDE from one particular test class by clicking on the test method and right click it. Or you can run all
test methods in a test class by right clicking on the test class name. 

If you want to run all unit test cases within a module, get into the module's directory and run.

```
mvn clean package
``` 

The above command will only run all unit tests without touching integration tests. 


There are several sub-projects in light-eventuate-4j are for testing only and only test classes are located
in these projects. 

For example, eventuate-client-test only contains src/test folder.

[unit test tutorial]: /tutorial/common/test/unit-test/
