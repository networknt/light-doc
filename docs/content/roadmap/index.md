---
date: 2016-10-15T19:19:57-04:00
title: Road Map
---

The short term goal is to make sure the API framework production ready.

# 1.3.0 Remove java from project names as it is a trademark of Oracle

# 1.4.0 Switch to HTTP/2 for client and server

# 1.5.0 Support API or Service Hosting

# 1.6.0 Create light-webserver
Requester: Eric Broda, October 9, 2017

Current web server components are relatively heavyweight (Apache) and
some are offer modest performance (NGINX).  All require complex configurations
to support secure operations, especially at-scale.  There is a market opportunity
to offer a light-weight, ultra high-performance, and secure web server
platform.  Our "light" framework addresses this market need:  it is ultra-high
performance, light-weight, and offers secure operations out-of-the-box.

# 1.7.0 Create light-gateway
Requester: Eric Broda, October 9, 2017

Current API gateway components are relatively heavyweight (CA/Layer7) and
some are offer modest performance (KONG).  All require complex configurations
to support secure operations, especially at-scale.  There is a general
market opportunity to offer a light-weight, ultra high-performance, and secure gateway
platform.  Our "light" framework addresses this market need:  it is ultra-high
performance, light-weight, and offers secure operations out-of-the-box.

However, the Internet-of-Things (IoT) market has explicit requirements for
these characteristics in particular.  This market is very large (hundreds of
billions of devices are forecast over next 10 years) and there is a market void
for this offering today.  

Ideally, this offering would target the "maker" movement and eventually
custom distributions for selected micro-platforms.  For example, by running
our framework on a Raspberry Pi but could easily be tailored for any
ARM processor running Linux.

# 1.8.0 Create light-router
Requester: Eric Broda, October 9, 2017

Current router components are relatively heavyweight and typically require
a hardware footprint.  All require complex configurations
to support secure operations, especially at-scale.  There is a market opportunity
to offer a light-weight, ultra high-performance, and secure software-based
router.  Our "light" framework addresses this market need:  it is ultra-high
performance, light-weight, and offers secure operations out-of-the-box.

# 1.9.0 Dynamic Loading of JARS
Requester: Eric Broda, October 9, 2017

Today, our framework requires a developer to generate / build all the full
stack of components (light-... plus their own business logic).  While not
necessarily cumbersome, it does require a build phase for components that
are part of the light-framework which will rarely change (at least relative
to the developer's business code).  

To address this issue, this initiative allows JARs to be loaded dynamically
based upon a simple configuration file (this config file would act in a
similar fashion to all other light-... configurations).  At startup,
a microservice will load all of its regular configurations as well as the
configuration files that specifies the JAR to load.  This configuration file
would map the JAR's functions to Swagger file end points which are resolved
at run-time.

# 1.10.0 Create PSD2 / Open Banking Demo Microservices

Requester: Eric Broda, October 9, 2017

PSD2 as well as the similar UK offering requires banks to provide open APIs
for customer, product, account, and transaction information.  These APIs are
referred to as "Open Banking APIs".  It is expected that
the Canadian market will require a "Open Banking APIs" in 2-3 years, and the US
shortly thereafter.  It is also expected that the Canadian and US initiatives
will create a similar model / framework to that "Open Banking APIs".

However, North American banks do not have a "starter kit" upon which to kick-start
their Open Banking initiatives.  To address this, this initiative offers
a starter kit for North American banks that will reduce costs and accelerate
delivery of Open Banking.

The "LIGHT Open Banking Starter Kit" will offer a set of microservices from
publicly available Open Banking Swagger specifications that connect to a
demonstration database (Cassandra, Postgres etc).

# 1.11.0 Compare our Framework to KONG Benchmark
Requester: Eric Broda, October 9, 2017

Today, KONG is a popular open source API gateway and management framework
built upon an NGINX foundation.  KONG offers a publicly available
benchmark (with instructions on reproducing the benchmark) which suggests
modest performance throughput.

This initiative will create a LIGHT benchmark that is identical to the KONG
benchmark and compare results.  We are confident we can dramatically exceed
KONG benchmarks and intent to publish the results, which will provide some
useful media attention and bragging rights.
