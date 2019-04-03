---
title: "Networknt Config"
date: 2019-03-25T13:11:51-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

We build the light-bot to first manage the numeric repositories in the networknt organization on GitHub. Also, we provide the tool to our users to manage their libraries and microservices with multiple repositories. 

For our own usage, we have all config files externalized to the [light-config-test][] repository. All configurations are scattered in several folders under light-bot folder. 

* checkout

This folder contains config files to allow users to check out all related repositories from networknt for their daily development work. It is recommended running it in the early morning to ensure that your local repositories are in sync with the remote GitHub repositories. The helps the developer to ensure they are working on the latest codebase.  

* create-branch

This folder contains config files to create maintenance branches for the networknt organization. Each time, we bump up minor and major version, we need to create a maintenance branch so that patches can be released there to support production upgrade for customers. 

* develop-build

This folder contains config files to support local build with tests and without tests. It is mainly used by the light-4j developers to build all related repositories in one shot to ensure that changes in one repository won't break other repositories. You can decide to run all unit, integration and end-to-end test cases or just run the build from your picked up branch to create the local dependencies in .m2 folder. 

* regex-replace

This folder contains config files to support regular expression replacement across all related repositories in the networknt organization. This is used to search and replace certain text in all repositories and push them to the remote Git server once it is done. For example, release a dependent library version for all repositories. 

* release-docker

This folder contains config file for docker release for all services within the networknt organization. These are all infrastructure services built by us, and most users are using them with Docker. For every maven release, we are releasing corresponding docker images at the same time.

* release-maven

This folder contains config file for maven release for all libraries within the networknt organization. These libraries are scattered in numeric repositories in networknt, and they are all depending on each other. They need to be built and tested in the right sequence. Once everything is built successfully, they will be pushed to maven central, and release notes will be pushed to the GitHub. 

* version-upgrade

Before every release in each release branch, we need to change the release number all together for all the related repositories. This task will run the maven version and replace the mutual dependency version within repositories. 

[light-config-test]: https://github.com/networknt/light-config-test

