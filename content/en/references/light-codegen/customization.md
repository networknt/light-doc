---
title: "Customization"
date: 2019-01-05T11:16:24-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


light-codegen is a generic code generator based [Rocker template engine][]. It can be extended to support other frameworks or other languages. You just need to package the new generator along with template files into a jar and put the jar file into the classpath of light-codegen to make it work. In order for the light-codegen to find your new generator, you need to externalize service.yml to add your generator implementation into it. The one used internally can be found at https://github.com/networknt/light-codegen/blob/master/codegen-fwk/src/main/resources/config/service.yml


[Rocker template engine]: https://github.com/fizzed/rocker

