---
title: "Contributing"
date: 2017-11-22T18:50:35-05:00
description: ""
categories: [contribute]
keywords: [dev,open source]
menu:
  docs:
    parent: "contribute"
    weight: 1
weight: 1
aliases: []
toc: false
draft: false
---

Third-party patches are essential for keeping Light platform great. We simply can't
access the huge number of platforms and myriad configurations for running
Light. We want to keep it as easy as possible to contribute changes that
get things working in your environment. There are a few guidelines that we
need contributors to follow so that we can have a chance of keeping on
top of things.

## Development

There are so many moving parts in light platform. Some are generic middleware handlers
or components to address cross-cutting concerns, some are handlers or components to
support certain style of interactions like REST, GraphQL and Hybrid/RPC. Some are parts
of the infrastructure services like light-oauth2/light-portal and some are parts of
the tool chain to support service development and deployment. 
 
If you are unsure of whether your contribution should be implemented in which repository
or which component, you may visit [Gitter Chat](https://gitter.im/networknt/light-4j) 
for advice. For more details on how to contribute to the codebase, please refer to
[development contribution][]. The following outlines the major steps.

#### Getting Started

* Make sure you have a [GitHub account](https://github.com/signup/free)
* Submit a ticket for your issue, assuming one does not already exist.
  * Clearly describe the issue including steps to reproduce when it is a bug.
  * Make sure you fill in the earliest version that you know has the issue.
* Fork the repository on GitHub

#### Making Changes

* Create a topic branch from where you want to base your work.
  * This is usually the develop branch.
  * Only target master branch if you are certain your fix must be on that
    branch.
  * To quickly create a topic branch based on develop; `git checkout -b
    fix32 develop`. Please avoid working directly on the `master` branch.
* Make commits of logical units.
* Check for unnecessary whitespace with `git diff --check` before committing.
* Make sure your commit messages are in the proper format. Start with "fixes #11" and 11 is the issue number you are trying to address.
* Make sure you have added the necessary tests for your changes.
* Run all the tests to assure nothing else was accidentally broken.


## Documentation

For changes of a trivial nature to comments and documentation, it is not necessary to create 
a new issue. The document site for light platform is https://www.networknt.com/ and each page
has a link to the github.com location in repo light-doc. For more details on how to contribute
documentation, please refer to [document contribution][]. The folowing outlines the major steps:


#### Getting started

* Make sure you have a [GitHub account](https://github.com/signup/free)
* It is no necessary to create a ticket to fix small issues in documentation; however, if it is a big change or adding new sections, it's better to have a ticket to track it.
* Fork light-doc repository on Github

#### Making Changes

* Push your changes to a topic branch in your fork of the repository.
* Submit a pull request to the light-doc repository in the networknt organization.
* The core team looks at Pull Requests on a regular basis and will merge the pull request. 
* Sometimes, we might need to contact contributor understand the details of the pull request.


## Example

Light platform contains so many frameworks and services in a big ecosystem. We cannot create
enough examples to cover every usage of the platform so every contribution on examples are
welcomed. For more details on how to contribute an example, please refer to [example contribution][].
The following outlines the major steps. 

#### Getting started

* Make sure you have a [GitHub account](https://github.com/signup/free)
* It recommended to create a ticket in light-example-4j before adding an example so that other people can learn more about the example. 
* Fork light-example-4j repository on Github
* Fork light-doc repository on Github


#### Making Changes

* Push your changes to a topic branch in your light-example-4j fork of the repository.
* Write a README.md in the folder of your example for other people to learn how to use your example.
* Update example section in light-doc fork repository to explain your example in detail.
* Submit a pull request to the light-example-4j repository in the networknt organization.
* Submit a pull request to the light-doc repository in the networknt organization.
* The core team looks at Pull Requests on a regular basis and will merge the pull request. 
* Sometimes, we might need to contact contributor understand the details of the pull request.

## Tutorial

Tutorials are very helpful for people to learn how to use the platform more productively. You
can write the tutorial on your own repository and write a blog post on your site. However, it
would be nice to have a link on our document site so that other people who are interested in
the light platform can easily find your article and source code. Also, if can include your
tutorial in the tutorial section of our document site and your source code in light-example-4j
for easy access by others.  

For more details on how to contribute a tutorial, please refer to [tutorial contribution][]. 
The following outlines the major steps if your code and article are part of our document site
and example repository.  

#### Getting started

* Make sure you have a [GitHub account](https://github.com/signup/free)
* It recommended to create a ticket in light-example-4j before adding a tutorial source directory so that other people can learn more about the tutorial. 
* Fork light-example-4j repository on Github
* Fork light-doc repository on Github


#### Making Changes

* Push your changes to a topic branch in your light-example-4j fork of the repository.
* Write a README.md in the folder of your tutorial for other people to learn how to use your tutorial.
* Update tutorial section in light-doc fork repository to explain your tutorial in detail.
* Submit a pull request to the light-example-4j repository in the networknt organization.
* Submit a pull request to the light-doc repository in the networknt organization.
* The core team looks at Pull Requests on a regular basis and will merge the pull request. 
* Sometimes, we might need to contact contributor understand the details of the pull request.


## Additional Resources

* [General GitHub documentation](https://help.github.com/)
* [GitHub pull request documentation](https://help.github.com/send-pull-requests/)
* [Light 4J Document](https://doc.networknt.com/)
* [Light 4J Gitter](https://gitter.im/networknt/light-4j)

[development contribution]: /contribute/development/
[document contribution]: /contribute/documentation/
[example contribution]: /contribute/example/
[tutorial contribution]: /contribute/tutorial/
