---
title: "Merge Branch"
date: 2019-03-27T14:05:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This task is used to merge from one branch to another for a list of repositories. 

Here are the steps in the task.

* it clones all projects defined in the checkout section into a workspace.
* if the workspace exists, then it will iterate each repository to switch to the branch specified in the checkout and pull from the remote. 
* merge the `from` branch to the `to` branch locally. 
* push the merged branch to the remote Git server

For live configuration example, please visit [light-config-test][]. 

Here is an example of the merge-branch.yml configuration file.

```
# This task is used to merge from one branch to another branch.
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: mergebranch
# The from branch name that is going to be merged to
from_branch: develop
# The to branch name that is going to be merged to
to_branch: master
# You can skip checkout if you are sure that the code in workspace are the latest and
# you just want to repeat the merge branch process due to some environmental issue before.
skip_checkout: false
# You can skip the merge branch step is this has been done in the previous execution.
skip_merge: false
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
    - git@github.com:networknt/light-spring-boot.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-example-4j.git
    - git@github.com:networknt/light-example-kotlin.git
    - git@github.com:networknt/light-doc.git
    - git@github.com:networknt/light-bot.git
    - git@github.com:networknt/light-docker.git
    - git@github.com:networknt/light-config-test.git
    - git@github.com:lightapi/light-config-server.git
    - git@github.com:networknt/microservices-framework-benchmark.git
    - git@github.com:networknt/model-config.git
    - git@github.com:networknt/light-workflow-4j.git
    - git@github.com:networknt/light-config-prod.git
```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/merge-branch/merge-all
