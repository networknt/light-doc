---
title: "Java Bean"
date: 2017-12-12T10:10:39-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Above use cases are about interface implementation and how to bind the implementation to
one or more interfaces. How about normal Java Beans? If there is a Java Bean and we want
to create a singleton instance that can be shared within the JVM. What to do in service
module. Further, some of the Java Beans have complicated initialization logic and where
to put the logic in? In this section, we are going to explore the POJO beans and how to 
utilize service.yml file to control how to initialize them as singletons. 

Let's assume we have an interface ChannelMapping. This use case has nothing to do with
interface anyway. 

```
public interface ChannelMapping {

    String transform(String channel);

}

```

And here is the default implementation for the above interface. 

```
public class DefaultChannelMapping implements ChannelMapping {

    private Map<String, String> mappings;

    public static class DefaultChannelMappingBuilder {

        private Map<String, String> mappings = new HashMap<>();

        public DefaultChannelMappingBuilder with(String from, String to) {
            mappings.put(from, to);
            return this;
        }

        public ChannelMapping build() {
            return new DefaultChannelMapping(mappings);
        }
    }
    public static DefaultChannelMappingBuilder builder() {
        return new DefaultChannelMappingBuilder();
    }

    public DefaultChannelMapping(Map<String, String> mappings) {
        this.mappings = mappings;
    }

    @Override
    public String transform(String channel) {
        return mappings.getOrDefault(channel, channel);
    }
}

```

In order to create an instance of DefaultChannelMapping, another bean IntegrationData is needed.

```
public class IntegrationData {
    private long now = System.currentTimeMillis();
    private String commandDispatcherId = "command-dispatcher-" + now;
    private String commandChannel = "command-channel-" + now;
    private String aggregateDestination = "aggregate-destination-" + now;
    private String eventDispatcherId  = "event-dispatcher-" + now;

    public String getAggregateDestination() {
        return aggregateDestination;
    }


    public String getCommandDispatcherId() {
        return commandDispatcherId;
    }

    public String getCommandChannel() {
        return commandChannel;
    }

    public String getEventDispatcherId() {
        return eventDispatcherId;
    }
}

```

To create these object, we will define a helper class to initialize them and we can invoke this
class from service module given the definition in service.yml

Here is the helper initializer class. 

```
public class ServiceInitializer {

    public IntegrationData integrationData() {
        return new IntegrationData();
    }
    public ChannelMapping channelMapping() {
        IntegrationData data = SingletonServiceFactory.getBean(IntegrationData.class);
        return DefaultChannelMapping.builder()
                .with("ReplyTo", data.getAggregateDestination())
                .with("customerService", data.getCommandChannel())
                .build();
    }

}
```

As you can see that there are two beans initialized IntegrationData and DefaultChannelMapping.
The later depending on the IntegrationData instance and get the singleton instance from the
SingletonServiceFactory. 

As there is dependency between these two beans, we need to define the IntegrationData before 
ChannelMapping in service.yml

Here is the section for these two beans. 


```
- com.networknt.service.IntegrationData: com.networknt.service.ServiceInitializer::integrationData
- com.networknt.service.ChannelMapping: com.networknt.service.ServiceInitializer::channelMapping

```

And here is the test case that calls these beans. 

```
    @Test
    public void testInitializerInterfaceWithBuilder() {
        ChannelMapping channelMapping = SingletonServiceFactory.getBean(ChannelMapping.class);
        Assert.assertNotNull(channelMapping);
        Assert.assertTrue(channelMapping.transform("ReplyTo").startsWith("aggregate-destination-"));
    }

```

In summary, this tutorial shows how to defined singleton beans and how to manage the dependencies
in service.yml file. Also, you can have complicate logic to initialize your beans in a helper
class. All the code in this tutorial can be found in the [service module test][]


[service module test]: https://github.com/networknt/light-4j/tree/master/service/src/test/java/com/networknt/service
