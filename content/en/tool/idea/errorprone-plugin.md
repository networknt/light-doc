---
title: "Error Prone Plugin"
date: 2019-04-24T16:53:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the light-4j 1.6.0 release, we have upgraded the Google error-prone plugin in the pom.xml to the latest version. This cause the IntelliJ IDEA to throw an error as the following. 

```
Error:java: java.lang.RuntimeException: java.lang.NoSuchMethodError: com.sun.tools.javac.util.JavacMessages.add(Lcom/sun/tools/javac/util/JavacMessages$ResourceBundleHelper;)V
Error:java: Caused by: java.lang.NoSuchMethodError: com.sun.tools.javac.util.JavacMessages.add(Lcom/sun/tools/javac/util/JavacMessages$ResourceBundleHelper;)V
Error:java: 	at com.google.errorprone.BaseErrorProneJavaCompiler.setupMessageBundle(BaseErrorProneJavaCompiler.java:202)
Error:java: 	at com.google.errorprone.ErrorProneJavacPlugin.init(ErrorProneJavacPlugin.java:40)
```

To make the IDEA work, we need to install a plugin. 

* Go to the Help/Find Action...
* Search Plugins
* Search `error` in the market place to find `Error-prone Compiler Integration`
* Install the plugin and restart the IDE
* Go to File | Settings | Compiler | Java Compiler and select `Javac with error-prone` in `Use compiler` box.

With the above changes, you can run and debug the application in the IDE. 

For more information about this plugin, please visit https://plugins.jetbrains.com/plugin/7349-error-prone-compiler-integration

