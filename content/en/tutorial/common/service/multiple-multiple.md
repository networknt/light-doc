---
title: "Multiple Multiple"
date: 2017-11-21T12:33:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is an complex use case that you have multiple interfaces and multiple implementations. It
is a rare use case but we need to know how to handle it.

Let's write two interfaces. 

```

public interface E {
    String e();
}

```

```
public interface F {
    String f();
}

```

Now let's write two implementations.

```
public class EF1Impl implements E, F {
    @Override
    public String e() {
        return "e1";
    }
    @Override
    public String f() {
        return "f1";
    }
}

```

```
public class EF2Impl implements E, F {
    @Override
    public String e() {
        return "e2";
    }
    @Override
    public String f() {
        return "f2";
    }
}

```

Write a service.yml to bind them together.

```
- com.networknt.service.E,com.networknt.service.F:
  - com.networknt.service.EF1Impl
  - com.networknt.service.EF2Impl

```


And here is an test case to test it.

```
    @Test
    public void testMultipleToMultiple() {
        E[] e = SingletonServiceFactory.getBeans(E.class);
        Arrays.stream(e).forEach(o -> System.out.println(o.e()));
        F[] f = SingletonServiceFactory.getBeans(F.class);
        Arrays.stream(f).forEach(o -> System.out.println(o.f()));
    }

```
