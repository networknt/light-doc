---
title: "Constructor Parameters"
date: 2017-11-21T12:33:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

As we are dealing with constructor injection and so far all our implementations have
a default constructor without any parameter. What if there are implementation with
parameters in constructor? This tutorial will show you how to do it. Also, the service
module supports constructor parameters to be injected as well. You just need to make
sure that parameters are injected early in the service.yml file.

Here is an interface.


```
public interface G {
    String getName();
    void setName(String name);
    int getAge();
    void setAge(int age);
}

```

And implementation

```
public class GImpl implements G {
    String name;
    int age;

    public GImpl() {
    }

    public GImpl(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String getName() {
        return name;
    }
    @Override
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public int getAge() {
        return age;
    }

    @Override
    public void setAge(int age) {
        this.age = age;
    }
}

```

Here is the service.yml binding. As you can see, we pass in two parameters to the 
implementation constructor. 

```
- com.networknt.service.G:
  - com.networknt.service.GImpl:
      name: Sky Walker
      age: 23

```

Here is the test case to use it.

```
    @Test
    public void testSingleWithProperties() {
        G g = SingletonServiceFactory.getBean(G.class);
        Assert.assertEquals("Sky Walker", g.getName());
        Assert.assertEquals(23, g.getAge());

    }

```
