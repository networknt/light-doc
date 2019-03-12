---
title: "OpenAPI 3.0 Specification"
date: 2019-03-11T22:31:38-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When you build a service based on the light-rest-4j framework, the first thing is to create an OpenAPI 3.0 specification. There are several options in the construction of the OpenAPI 3.0 specification. 

### Online Tool

If you are dealing with small services, you can use the online tool [swagger editor][] to create specifications. 

### Editor and Bundler

For large organizations that are planning to build hundreds or thousands of services, it is wise to share some components across multiple services. It requires to split a large specification into smaller sharable pieces during the development. Each service will have its specification which contains references to the sharable spec files. 

You can use the above only tool to edit the specification file; however, be aware that it is very slow for large spec files. A lot of our customers are using a plain editor to author the spec and then use [OpenAPI bundler][] to produce a final specification file per service with all the externalized references resolved locally. 

[swagger editor]: https://editor.swagger.io/
[OpenAPI bundler]: https://github.com/networknt/openapi-bundler
