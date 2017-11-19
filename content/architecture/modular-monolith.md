---
title: "Modular Monolith"
date: 2017-11-11T17:20:42-05:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "architecture"
    weight: 10
weight: 10
aliases: []
toc: false
draft: false
---

For some people microservices is "the next big thing", whereas for others it's simply a lightweight 
evolution of the big service-oriented architectures that we saw 10 years ago "done right". 
Microservices is by no means a silver bullet though, and the design thinking required to create a 
good microservices architecture is the same as that needed to create a well structured monolith. 
And this begs the question that if you can’t build a well-structured monolith, what makes you think 
microservices is the answer?

If it is not designed correctly, microservices is also known as a distributed big ball of mud. And
people always assume that all monoliths are big balls of mud. 

MSAs, like any distributed application architecture, involve increased complexity and costs; table 
stakes, if you will. Like an iceberg, there’s both a lot more to it than just what’s showing above 
the waterline and a fair amount of hazard for the unwary. If a development team cannot or will not 
comply with design guidelines (e.g. modularity requirements), injecting additional complexity is 
probably not the solution you need.

Distributing an application makes it harder to accidentally entangle different concerns, but it 
doesn’t make it impossible. 

I’d argue that making it harder to accidentally break modularity addresses neither of the groups 
I mentioned earlier: those that cannot or will not comply. It’s ironic, but those who fail to 
understand the need for modularity can be very creative in their “solutions”, regardless of the 
obstacles. In short, distribution as a means of “ensuring” modularity fails the fitness for 
purpose test.

The situation becomes worse when you factor in the additional complexity inherent in a distributed 
system. Likewise, there’s the cost of the table stakes (infrastructure, process, staffing, etc.) 
mentioned above. Of course, having abandoned the principle of cause and effect, one could attempt 
some “creative” workarounds to avoid having to pay the price (in other words, adding more and more 
complexity).

When you introduce significant additional complexity (with all its attendant risk) with little 
chance of the technique actually achieving its goal, you’ve caused harm.

### Why are microservices useful?

Microservices are useful because of two things:

* scale up the team very quickly by taking advantage of Conway’s Law 

* modular development

In addition, microservices can scale very easily and minimize spending on the cloud. The reason is 
that when load increases, more specific instances can be started at a minimal cost.

As we can see from the industry examples, these criteria apply very well for large projects that 
need to scale quickly. So if you need to start such a project, by all means, use microservices. 
But if you don’t need to scale that quickly, there are easier ways. The reason is that 
microservices, like any architectural approach, are a trade-off.

### A downside and a huge mistake

Microservices have one important downside and can lead to one huge architecture mistake.

The downside is that microservices architecture leads to huge complexity increases for deployment 
and operations. Deploying one monolith is very different from deploying 100 or 1000 separate services. 
Monitoring and de-bugging then become a very difficult job. There are solutions, but each adds to 
costs and complexity. This investment might not pay off for smaller products.

The architecture mistake is to create microservices that directly depend on other microservices. 
This is just a re-creation of the dependency hell problem in deployment. If dependencies are 
hierarchical, changing a service propagates and require a lot of redeployments. Instead, aim for 
services that completely encapsulate one atomic change. This is the hardest thing to do about 
microservices; no wonder it took Fred George’s team weeks to figure it out (and probably a lot 
more time after starting the development).

### Modular monolith

There is an alternative to microservices, one that we used successfully for light-hybrid-4j: the 
modular monolith.

Can we realize the benefit of microservices and still enjoy the simplicity of monolith? The answer
is modular monolith. It is an architecture sitting in between monolith and microservices. Instead of
big black box of monolith, we apply [microservices principles][] to vertically slice the application
to multiple independent services; however, we don't want to deploy these services distributed in the
first place but put them together so that the communication is in-process instead of HTTP or other
network protocol. This removes the complex topology of microservices and hugo front infrastructure
cost. And it leave the room to scale these services individually if load is heavy. 

In JVM, this means we have several independent services deployed into the same JVM whether or not
they are in the same container or virtual machine. 
 

[light-hybrid-4j] framework is designed to be a modular monolith or microservices framework. Developers
can build services just like the normal microservices with the final output a jar file. Several jar
files can be deployed to the same docker container volume and loaded into the same JVM. The communication
between these services are through well defined interface contract and all the details for each service
is hidden. If one or more services are getting too much load, then you can put these services into
separate containers to scale them only. 

[microservices principles]: /architecture/microservices/
[light-hybrid-4j]: /style/light-hybrid-4j/


