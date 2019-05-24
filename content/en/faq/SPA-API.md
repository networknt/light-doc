---
title: "SPA API"
date: 2017-11-06T16:10:53-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

### How to serve Single Page Application(React/Angular etc.) from your API server instance


During development, we normally use Nodejs to serve the single page application as it is easy
to package and test/debug. However, on production, it makes sense that the API server serves
the single page application by itself. In this way, we don't need to enable [CORS handler][]
and same another instance of the server.

To learn how to use Nodejs for Single Page Application development, please take a look at
https://github.com/networknt/light-codegen/tree/master/codegen-web

Here I will give you an example to show you how to package your application and deploy it
with the API built on top of light-*-4j frameworks.

There are two options depending on if you are using Docker or not.

### Without Docker

You can serve the SPA with command line to start the server. The only thing that you need
to make sure is to have the webserver.json config file point to the exact local filesystem
directory. The default config point to the relative folder which is not going to work.

For more info, please take a look at the [webserver example][] and its readme.

### With Docker

It is much easier to deploy SPA with containerized API as you can just put your SPA into
a folder in your host machine and then map into the Docker with a volume. The API server
can find the static files and serve them. Also, it allows you to change the UI easier without
touching the API.

Take a look at a Dockerfile in [webserver example][] example and pay attention on the volume that maps 
a host folder to a folder in the container.

[CORS handler]: /concern/cors/
[webserver example]: https://github.com/networknt/light-example-4j/tree/develop/webserver
