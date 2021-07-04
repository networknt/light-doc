---
title: "How light-bot works"
date: 2019-03-25T10:34:18-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The light-bot is a command line tool at the moment. We can easily wrap it with a light-4j server to enable it as a service in the future. 

The command line entry point is the `Cli` class in the `bot-cli` module. When invoking the command line, you need to specify the task name you want to execute. Also, you need to specify the config folder for the task configuration files. 

The light-bot is based on light-4j service module for dependency injection so almost every class is a plugin. It also leverages the config module of light-4j so that each plugin has its configuration files to control the behavior. 

There are two types of classes: command and task. 

A command is a fine-grained action that focuses on only one specific job, for example, creating a branch for a Git repository. It has its configuration on how the action should be taken, but usually, they don't work independently. All commands are located in the exec-core module cmd folder. 

A task is a group of commands glued together with a configuration file to define the sequence and logic for the command executions. For example, the create-branch task will call the following commands to complete. 

* Check out all repositories
* For each repository, create a new branch
* For each repository, push the new branch to the Git server.

Each task has a corresponding module in the light-bot. For advanced users, they can create new tasks and package them into a jar file. To plug in your external jar file in light-bot, just put the jar into the classpath in the command line and update the externalized service.yml to map your class. 

Here is a list of tasks available now. 

* create-branch

Create a new branch from a tag or another branch for a list of repositories and push to the Git server. We are using it to create maintenance branches once we increase the minor version for the networknt organization. 

* develop-build

Check out a list of repositories, build them with maven and test them by starting the servers with real requests. This is the most used task by our developers to sync and build all light-4j modules into the local .m2 repository or to perform integration tests after some services are changed.  

* merge-branch

This is a task that allows us to merge multiple repositories from one branch to another and push the merged result to the remote. 

* regex-replace

This task is used to do regular expression replacement for multiple repositories. We are using it to upgrade certain dependencies in all networknt repositories. It is better than the full-text search and replacement as the task can check out and check in multiple repositories automatically based on the regex-replace.yml config. 

* release-docker

This task is used to check out multiple repositories, build, test and push the final docker images to the docker hub. We are using it to publish light-4j infrastructure services like light-oauth2, light-proxy, and light-router, etc. 

* release-maven

This is the task we use to release libraries to the maven central for the light-4j platform. It checks out all related repositories, builds, tests, publishes to maven central, generates changelog and creates a release notes on the GitHub.


* version-upgrade

After each release, we need to use this task to upgrade all related repositories to a new version. This task also updates the dependencies in each project to ensure that all repositories have the right dependencies. 

