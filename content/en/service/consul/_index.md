---
title: "Consul"
date: 2018-08-01T09:47:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

There are numeric options for service registry and discovery in Light; however, Consul is the most important one as almost all our customers are using it. For development purpose, you can use Docker to start a single instance Consul server. The command line can be found in the [discovery tutorial][].

For production, the deployment is more complicated. If you have multiple data centers, you'd better use the enterprise edition and let HashiCorp install it. For simple cloud deployment, you can follow the [consul tutorial] to install a three nodes cluster. When using Light, there is no need to install the agent to the service host. A three nodes cluster of Consul server would be enough to support hundreds or thousands of service instances. 

[consul checklist]: /tool/consul/checklist/
[discovery tutorial]: /tutorial/common/discovery/consul/
[consul tutorial]: /tool/consul/cluster-install/
