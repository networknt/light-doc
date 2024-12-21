---
title: "Create Branch"
date: 2019-03-27T14:05:20-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This task is used to create a new branch for a list of repositories from an existing branch or an existing tag. 

Here are the steps in the task.

* it clones all projects defined in the checkout section into a workspace.
* if the workspace exists, then it will iterate each repository to switch to the branch specified in the checkout and pull from the remote. 
* create a new branch locally from an existing the current checked out branch or an existing tag. 
* push the newly created branch to the remote Git server

For live configuration example, please visit [light-config-test][]. 

Here is an example of the create-branch.yml configuration file.

```
# This task is used to create a new branch from an existing branch or a tag.
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: createbranch
# The new branch name that is going to be created
branch: 1.5.x
# If the new branch is created from a tag
from_tag: false
# The original tag to create a new branch if from_tag is true.
tag: 1.5.32
# You can skip checkout if you are sure that the code in workspace are the latest and
# you just want to repeat the create branch process due to some environmental issue before.
skip_checkout: false
# You can skip the create branch step is this has been done in the previous execution.
skip_branch: false
# You can skip the git push after the branch is created. The default is push to the git server.
skip_push: false
# clone and switch to the branch specified in checkout step.
checkout:
  - branch: master
    repository:
    - git@github.com:networknt/light-4j.git
    - git@github.com:networknt/json-overlay.git
    - git@github.com:networknt/openapi-parser.git
    - git@github.com:networknt/light-rest-4j.git
    - git@github.com:networknt/light-graphql-4j.git
    - git@github.com:networknt/light-hybrid-4j.git
    - git@github.com:networknt/light-codegen.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-kafka.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-saga-4j.git
    - git@github.com:networknt/light-session-4j.git
    - git@github.com:networknt/light-spa-4j.git
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-tokenization.git
    - git@github.com:networknt/light-consumer-4j.git
    - git@github.com:networknt/light-example-4j.git
    - git@github.com:networknt/light-example-kotlin.git
    - git@github.com:networknt/light-docker.git
    - git@github.com:lightapi/light-config-server.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-workflow-4j.git
    - git@github.com:networknt/http2client-benchmark.git
    - git@github.com:networknt/light-spring-boot.git
```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/create-branch/1_5_x

