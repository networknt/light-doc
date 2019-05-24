---
title: Multi Static Handler Chains
linktitle: Multi Static Handler Chains
date: 2017-10-17T19:41:50-04:00
lastmod: 2018-05-15
description: "Configuring your microservice to support multiple static handler chains to define exactly which http handlers are invoked for a specific request."
weight: 50
sections_weight: 50
draft: false
toc: true
---

## Introduction

The structure to define your handlers configuration requires a few definitions:

`handler`: An instance of the undertow `HttpHandler` class. To help save on typing and increase readability
you have the option to name your handlers with the `class@name` format. You will see an example of this later on.

`chain`: A named list of handlers, included to reduce repetition within the configuration.

`exec`: The execution chain to which a request will flow for a given path and method pair.

## Handlers 

The first thing you will need to do within your config is define a complete list of all the handlers that will
be used in your microservice. It could look something like this:

```yaml
handlers:
  - com.networknt.handler.sample.SampleHttpHandler1
  - com.networknt.handler.sample.SampleHttpHandler2
  - com.networknt.handler.sample.SampleHttpHandler3@third
```

In this case we define 3 handlers, the first two being named by their fully qualified class, where as the
last will be referenced by the given name `third`.

You will need to know and use the names of the handlers when defining reusable chains and execution lists.

It is important to note also, that each of the handlers within this list will be a singleton. So no matter
how many times it's referenced throughout the configuration, only once instance of the object is ever created.
If you needed multiple instance of the same class, you have the option to name them differently and reference 
them as such.

## Chains

A chain defines an ordered list of handlers that can be referenced under a single name. This option 
is solely introduced as a means to save typing and increase readability.

To define a named chain, use the following format within the configuration:

```yaml
chains:
  secondBeforeFirst:
    - com.networknt.handler.sample.SampleHttpHandler2
    - com.networknt.handler.sample.SampleHttpHandler1
  thirdBeforeSecond:
    - third
    - com.networknt.handler.sample.SampleHttpHandler2
```

As you can see, we reference the chains by the given name we provided in the `handlers` section.
If you didn't explicitly provide a name, the fully qualified class is used instead.

## Paths

Paths define the execution that will occur as a request from the client is issued. They require 3 fields
to be defined:

`path`: The url path of the request.

`method`: The HTTP request method issued to this path.

`exec`: The execution list that will be triggered when the request is received.


For example:

```yaml
paths:
  - path: '/test'
    method: 'get'
    exec:
      - secondBeforeFirst
      - third
  - path: '/v2/health'
    method: 'post'
    exec:
      - secondBeforeFirst
      - third
```

In this case, our microservice is supporting 2 request paths, one is a `GET` call at the `/test` endpoint
and the other a `post' call on the `/v2/health` endpoint.

To expand on the execution that will occur when calling the `/test` end point, we can see that it first 
references the `secondBeforeFirst` chain, followed by the handler named `third`. So ultimately the call will be:
```yaml
- com.networknt.handler.sample.SampleHttpHandler2
- com.networknt.handler.sample.SampleHttpHandler1
- com.networknt.handler.sample.SampleHttpHandler3@third
```

Note: If the names of a chain and a handler ever conflict, the name of the chain will be used.


## Usage

To maintain backwards compatibility, not including a `handler.yml` (or including one with `enabled: false`)
will cause the previous structured config to be used instead (ie a single execution flow for all requests, 
defined in service.yml).

However to include it, the previous service.yml config for middleware will be ignored.

Usage is defined by including the `handler.yml` within your `resources` or `config` directory (or externalized
and passed in), and setting `enabled: true`.

## Complete Sample

```yaml
enabled: true

handlers:
  - com.networknt.handler.sample.SampleHttpHandler1
  - com.networknt.handler.sample.SampleHttpHandler2
  - com.networknt.handler.sample.SampleHttpHandler3@third

chains:
  secondBeforeFirst:
    - com.networknt.handler.sample.SampleHttpHandler2
    - com.networknt.handler.sample.SampleHttpHandler1

paths:
  - path: '/test'
    method: 'get'
    exec:
      - secondBeforeFirst
      - third
  - path: '/v2/health'
    method: 'post'
    exec:
      - secondBeforeFirst
      - third
```