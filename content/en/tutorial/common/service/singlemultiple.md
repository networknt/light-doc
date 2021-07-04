---
title: "Single Interface Multiple Implementations"
date: 2017-11-21T12:09:48-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is a use case where you have an interface, and multiple implementations will all be used
during runtime. For example, in light-4j framework, we have multiple StartupHookProviders to
be loaded during server startup and execute them one by one. Also, all our middleware handlers
are wired into the request/response chain the same way. 

Let's say we have this interface. 

```
public interface Processor {
    String process();
}

```
 
And we have the following three implementations want to be executed in sequence. 

```
public class ProcessorAImpl implements Processor {
    @Override
    public String process() {
        return "A";
    }
}

```

```
public class ProcessorBImpl implements Processor {
    @Override
    public String process() {
        return "B";
    }
}

```
 
 
```
public class ProcessorCImpl implements Processor {
    @Override
    public String process() {
        return "C";
    }
}

```

Here is the service.yml configuration. 

```
singletons:
- com.networknt.service.Processor:
  - com.networknt.service.ProcessorAImpl
  - com.networknt.service.ProcessorBImpl
  - com.networknt.service.ProcessorCImpl

```

And here is the test case to show how to use it.

```
    @Test
    public void testGetMultipleBean() {
        Processor[] processors = SingletonServiceFactory.getBeans(Processor.class);
        Assert.assertEquals(processors.length, 3);
        Arrays.stream(processors).forEach(processor -> System.out.println(processor.process()));
    }

```
