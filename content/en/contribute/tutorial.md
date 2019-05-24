---
title: "Tutorial"
date: 2017-11-04T09:35:13-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "contribute"
    weight: 70
weight: 70
aliases: []
toc: false
draft: false
---

The light platform is aiming enterprise-scale microservices, and there are too many moving parts. The general feedback from our users is that it is hard to learn especially for Java EE developers. Even there are a lot of examples in [light-example-4j][] repository and we have so many infrastructure services built on the platform like [light-oauth2][], [light-portal][], [light-proxy][] and [light-router][], a lot of beginners still don't know where to get started. 

To help our users, we have written tutorials that cover almost every perspective of the platform. These tutorials can be found at [document site][] and the repository is [light-doc][].

If you are contributing examples in light-example-4j, instead of writing a README.md to introduce your project, it is highly recommended that you write a tutorial for your project. In the process, you will learn more as you have to rethink all the details. For other people who are good writers, we need your help to write tutorials based on examples or write blogs about your experiences in using the platform. 

### Getting started

* Make sure you have a [GitHub account](https://github.com/signup/free)
* It recommended to create a ticket in light-example-4j before adding a tutorial source directory so that other people can learn more about the tutorial. 
* Fork light-example-4j repository on Github
* Fork light-doc repository on Github


### Making Changes

* Push your changes to a topic branch in your light-example-4j fork of the repository.
* Write a README.md in the folder of your tutorial for other people to learn how to use your tutorial.
* Update tutorial section in light-doc fork repository to explain your tutorial in detail.
* Submit a pull request to the light-example-4j repository in the networknt organization.
* Submit a pull request to the light-doc repository in the networknt organization.
* The core team looks at Pull Requests on a regular basis and will merge the pull request. 
* Sometimes, we might need to contact contributor understand the details of the pull request.


[document site]: /tutorial/
[light-doc]: https://github.com/networknt/light-doc
[light-example-4j]: https://github.com/networknt/light-example-4j
[light-oauth2]: /service/oauth/
[light-proxy]: /service/proxy/
[light-router]: /service/router/
[light-portal]: /service/portal/
