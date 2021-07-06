---
title: "Firewall"
date: 2017-12-14T09:42:12-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The traditional firewall is dead or at the very least dying.

Cloud and hybrid environments, mobile access, and online applications have made it all but 
obsolete, experts say, and data center operators should be looking at replacing their firewalls with more granular security technologies.

Applications and data used to live in data centers and be delivered from there to employees 
who were themselves on a corporate network with a desktop on a static IP address.

“Even when off-site, they were VPNing into the network,” said Michael Beesley, CTO at Skyport 
Systems.

That’s not true anymore. Today, applications can live in cloud and hybrid environments or be 
delivered via websites by external services providers. Employees and customers access them, often via the web, from wherever they happen to be.

That means the firewall can’t see what’s going on, where the connections are coming from, or 
where they’re going, while the IP addresses change all the time or are obscured by content 
delivery networks like CloudFlare.

“The firewall becomes blind,” he said.

Meanwhile, the traffic that used to be distributed among many different ports is now all 
concentrated on port 80 and 443.

“I think we’re approaching a tipping point where out of necessity enterprises are going to 
adopt a different approach with regard to how they secure themselves,” Beesley said. “The 
empirical data is that in a hybrid environment the firewalls are not doing their job. 
Infrastructure needs to evolve to a zero-trust environment rather than trying to secure it 
from a networking point of view.”

Then there’s the encryption issue.

According to Google, 79 percent of traffic in the Chrome browser is currently encrypted.

“The browser is what killed the firewall,” said Ryan Spanier, director of research at 
Kudelski Security. “Because you had clients asking for things on the internet, and the 
firewall wouldn’t stop a thing.”

With encrypted traffic, all the firewall knows is the source and the destination of the 
traffic, and now that everything is in the cloud, even that doesn’t tell you much. “There’s 
not a lot of room for the firewall anymore,” he said.

According to a survey released this month by network security vendor Ixia, 88 percent of 
respondents said they experienced a business-related issue from a lack of visibility into 
public cloud data traffic.

Finally, there’s the transformation of application development from monolithic to microservices. 
Enterprises have moved from running applications on dedicated servers to virtual machines to 
containers, each time dramatically increasing the number of endpoints that need to be protected 
while simultaneously accelerating the rate at which new ones are spun up and shut down.

Given we cannot rely on network security or firewall to protect our services, then what is
the solution? Unlike the old world you have monolithic applications, if you have hundreds or
thousands of microservices deployed in the cloud with some well known interface like REST,
then everyone can access it if there is no authentication or authorization. 

In Light, we are moving the security to the application itself as part of the framework
and all each individual service instance to enforce the security policy defined on light-oauth2
server. In another words, centralized policy management but decentralized policy enforcement.

The reason all vendors trying to provide commercial offering for secuity are based on the network
layer is due to lack of knowledge on the application itself or lack of access to the application.

As we build the [security handler] as a middleware handler and wire it into the request/response
chain of your service, it intercepts the request and have the clear text data as TLS termination
is done already. It can make the decision on if the request is authorize or not based on two 
different layers of security. 

### Technical Layer of Security

This layer of security is based on OAuth 2.0 and is the same for all kind of services regardless
the business domain. It is implemented in the light-4j and related frameworks and ready to be
plugged in. It uses JWT tokens and validate if the client_id in the token has access to the service
and if the scope in the token has access to the endpoint it is calling. So the authorization level
is at the endpoint level in this layer.

### Business Layer of Security

The is the next layer of security after the technical layer and it must be run within the business
context. If the previous layer of security is passed and the request has access to the endpoint like
fund transfer, there are certain authorization rules that need to be executed in order to authorize
the request. For example, if original user is just a teller, he/she can only transfer less than $10K,
if the user is based at Asia, then he/she cannot transfer out fund for customers based at America.

As you can see this layer of security is business related and cannot be handled at technical level
in a generic way. In Light, we are providing a generic handler framework and our customers
can fill in the business rules to define fine-grained authorization. 

[security handler]: /concern/security/


