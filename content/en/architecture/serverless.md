---
title: "Serverless"
date: 2017-11-11T17:19:01-05:00
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


Serverless is a hot topic in the software architecture world. We’re already seeing books, open source 
frameworks, plenty of vendor products, and even a conference dedicated to the subject. But what is 
Serverless and why is (or isn’t) it worth considering? 

To start we'll look at the ‘what’ of Serverless where I try to remain as neutral as I can about the 
benefits and drawbacks of the approach - we'll look at those topics later.

### What is Serverless?

Like many trends in software there’s no one clear view of what ‘Serverless’ is, and that isn't helped 
by it really coming to mean two different but overlapping areas:

* Serverless was first used to describe applications that significantly or fully depend on 3rd party 
applications / services (‘in the cloud’) to manage server-side logic and state. These are typically 
‘rich client’ applications (think single page web apps, or mobile apps) that use the vast ecosystem 
of cloud accessible databases (like Parse, Firebase), authentication services (Auth0, AWS Cognito), 
etc. These types of services have been previously described as ‘(Mobile) Backend as a Service’, and 
I’ll be using ‘BaaS’ as a shorthand in the rest of this article.

* Serverless can also mean applications where some amount of server-side logic is still written by 
the application developer but unlike traditional architectures is run in stateless compute containers 
that are event-triggered, ephemeral (may only last for one invocation), and fully managed by a 3rd party. 
One way to think of this is ‘Functions as a service / FaaS’ . AWS Lambda is one of the most popular 
implementations of FaaS at present, but there are others. I’ll be using ‘FaaS’ as a shorthand for 
this meaning of Serverless throughout the rest of this article.

### Benefits

#### Reduced operational cost

* Infrastructure cost

* People (operation/development) cost


#### BaaS - reduced development cost

#### FaaS - scaling costs

* occasional requests

* inconsistent traffic

* Optimization is the root of some cost savings

#### Easier Operational Management

* Scaling benefits of FaaS beyond costs

* Reduced packaging and deployment complexity

* Time to market / experimentation

 
### Drawbacks

#### Inherent Drawbacks

* Vendor control

* Multitenancy Problems

* Vendor lock-in

* Security concerns

* Repetition of logic across client platforms

* Loss of Server optimizations

* No in-server state for Serverless FaaS

#### Implementation Drawbacks

* Configuration

* DoS yourself

* Execution Duration

* Startup Latency

* Testing

* Deployment / packaging / versioning

* Discovery

* Monitoring / Debugging

* API Gateway definition, and over-ambitious API Gateways

* Deferring of operations

### The future of Serverless



### Conclusion

