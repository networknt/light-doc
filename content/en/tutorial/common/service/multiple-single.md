---
title: "Multiple Single"
date: 2017-11-21T12:33:27-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This is a use case that we have multiple interface but only need to load one implementation
during runtime. 

Here are two interfaces.

```
public interface D1 {
    String d1();
}

```

```
public interface D2 {
    String d2();
}

```

And here is an implementation.

```
public class DImpl implements D1, D2 {
    @Override
    public String d1() {
        return "d1";
    }
    @Override
    public String d2() { return "d2"; }
}

```

Now we need to have a service.yml file to bind them together.

```
- com.networknt.service.D1, com.networknt.service.D2:
  - com.networknt.service.DImpl

```

To write a test case for it.


```
    @Test
    public void testMultipleInterfaceOneBean() {
        D1 d1 = SingletonServiceFactory.getBean(D1.class);
        D2 d2 = SingletonServiceFactory.getBean(D2.class);
        Assert.assertEquals(d1, d2);
    }

```
