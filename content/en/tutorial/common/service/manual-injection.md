---
title: "Manual Injection"
date: 2017-12-12T10:10:51-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

So far, all singleton instances are created/loaded from service.yml and this file is pre-defined
before server startup. What if someone wanted to inject a bean into the service map directly in the
code? This is particularly useful during testing as you can switch beans or implementations as you
wish at anytime during the execution. It is however very hard for people to reason about in the
production code, so it is definitely not recommended to be used in this specific scenario. 

Let's assume you have a bean called InjectedBean

```
public class InjectedBean {
    public String name() {
        return "Injected Bean";
    }
}

```

And you want to inject it into the JVM service map in the BeforeClass in your test. Later, you will use it in your test cases. 



```
    @BeforeClass
    public static void setup() {
        InjectedBean injectedBean = new InjectedBean();
        SingletonServiceFactory.setBean(InjectedBean.class.getName(), injectedBean);
    }

    @Test
    public void testInjectedBean() {
        InjectedBean injectedBean = SingletonServiceFactory.getBean(InjectedBean.class);
        Assert.assertEquals("Injected Bean", injectedBean.name());
    }

```

As you can see, the bean is injected using setup and the instance is retrieved in one of 
the test case. 

The above API setBean is very convenient for testing but is not supposed to be on production, 
as people cannot reason about where the bean coming from. This is especially true if you replace some
beans defined in service.yml with something else in your code. Nobody can tell which instance
of the class is in used without debugging into it. The other benefit of using service.yml
is that you can change it on production through configuration change in order to swap some
implementations. 

