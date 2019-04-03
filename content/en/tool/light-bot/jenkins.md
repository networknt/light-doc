---
title: "Jenkins and Microservices"
date: 2018-02-07T14:27:01-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When we move to microservices architecture, there are too many repositories and dependencies
need to be managed by the DevOps tool and traditional monolithic focused tool like Jenkins
gives us a lot of hard time to make it work. I like Jenkins and I have been using it since its 
early days and I agree that Jenkins 2 is much better in designing pipelines. However, this time
it looks like we cannot use it for our networknt organization and our customers who are doing
microservices. Here is a list of issues we were facing and some of them might be resolved already 
and I welcome comments and corrections from the community.


* Pipeline cannot handle multiple repos

Even with Jenkins 2.x pipeline support, it is very hard to config multiple repos to be built
together. One use case in github networknt organization, we have about 46 repositories and about
20 of them are related. For example, if light-4j is change, all other 20 repositories need to
be built and tested with the new version of light-4j in develop branch. Another use case is for
microservices application that consist of dozens of smaller services. If one of the service is
changed, we need to do the integration test that will involve upstream and downstream services
with the latest code in different branches and simulate different environments(different test
data sets or different databases). 

* Doesn't support microserivces integration test

It cannot support integration tests with multiple services at once with single pipeline. Not
mention we have multiple environments and different configuration combinations that need to
be tested. For example, we need to test multiple services with different data sets provided
by different LOBs. We need to test these services deployed in data center with java processes,
docker-compose in a VM or Kubernetes cluster. We also need to test with different configuration
and observe the behaviour change accordingly. 

* Jenkins can only clone Git repository

As we are dealing with too many repositories in each build, we need to pull the git repository
instead of cloning it each time. It takes about 5 minutes to clone all networknt repositories on
a relative fast cable network and pulling repositories takes about 5 seconds to get workspace ready.
While we wish to run automated build at every commit, the issue compounds itself several times over.

* It is very hard to share the workspace

As different repositories are separate builds and they don't share the .m2 local maven repository.
That make each build very slow and generate too much network traffic. Also, before the previous
module is deployed, it is very hard to be shared when building the depending modules. 

* Too many plugins

There are about 1500 plugins provide functions for Jenkins and even basic tasks like checkout
from github or create docker image needs a plugin. Most of the plugins are developed by third
parties with different quality and support. For the same task, there might be several plugins
to choose from and chances are the one you have chosen lost support without notice. Building 
a software delivery chain based on third-party plugins is not a good way to ensure availability 
or stability.

* Not Docker and Cloud friendly

It is based on old technology in early 2000s - long before containers, microservies, cloud
native as the infrastructure of choice. It integrate with Docker via multiple plugins and most
of them are specific for vendor platforms. These plugins are very hard to do any customization
with the specific infrastructure environment our customers have to deal with. In my opinion,
it was designed for bare metal and virtual machines. Docker and Kubernetes are just something
plugged in and not integrated gracefully. 

* Jenkins is a CI tool but not CD

If Jenkins cannot do integration test and end-to-end test efficiently for microservices, you
can imaging that it is not a tool for production delivery neither. The rigid pipeline cannot
handle the dynamic cloud native environment and all the configuration management with related
services as part of the dependency tree. In order to have release automation, the tool needs
to provide communication tools and channels to enable the software delivery and collaboration
seamlessly. The DevOps tool chain needs to tightly integrated with your infrastructure services
and the microservice frameworks. 

* It is too slow and not scalable
   
Given it is built on JavaEE stack, it is not scaling very well. In one organization that have
hundreds developers building APIs, we have seen our release tasks in the queue for hours waiting
for available worker. In microservices world, if we have thousands services and some of the
services need to be updated, tested and released on a daily basis, I cannot image it can handle
the load. 

* Developer or branch build support is lacking

It is used as an official building tool and lack of flexibility to support multiple branch build
and integration tests inside and outside of developers' desktop or laptop. For example, if the
service built depending on Oracle database, developers cannot test it on his/her laptop, there
must be a way that he/she can work on their git branch and changes can trigger a build to test
on a build server. Also, it doesn't provide support for developer tasks as it can only run on 
the server not developers' laptop. For example, if the API framework version is upgraded, can
a developer run a task on his local machine to update 18 APIs with the latest framework version?
The Git branching model adopted for one of our customers is based on at least 2 standard branches 
running in parallel at all times; develop and release. Adding to this feature branches, issue 
becomes N*M. 

In summary, I think Jenkins is a monolithic application built as server centric approach and
only support horizontal scaling. When we adopt microservice, architecture, should we start
thinking our DevOps tool chain as microservices? And it can run anywhere and everywhere to
to vary tasks suitable only for that environment? If it is fully distributed and customized
just like building Lego structures. Then We can make our DevOps on developers desktop, data
center, cloud etc. and it scales infinitely. 

  
 


