---
title: "Single Interface Single Implementation"
date: 2017-11-13T22:32:05-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is a very common use case that you have an interface and during runtime, you want
to inject one and only one instance of implementation. 

Let's say here is an interface. 

```
public interface A {
    String a();
}
```

And we have an implementation like this.

```
public class AImpl implements A {
    @Override
    public String a() {
        return "a real";
    }
}
```

Now we need to put service.yml in src/main/resources/config folder.

```
singletons:
- com.networknt.service.A:
  - com.networknt.service.AImpl

```

This is a configuration file that binds com.networknt.service.AImpl implementation
to com.networknt.service.A interface. 

Let's see how to use it in a test case. 

```
    @Test
    public void testGetSingleBean() {
        A a = SingletonServiceFactory.getBean(A.class);
        Assert.assertEquals("a real", a.a());
    }

```
