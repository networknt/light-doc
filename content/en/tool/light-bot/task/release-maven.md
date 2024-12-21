---
title: "Release Maven"
date: 2019-03-27T14:04:52-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

This is the task to release all frameworks and modules all together for the networknt organization on the GitHub. It first checkout repositories into a workspace and switch to the release branch defined in the release-maven.yml file. If the workspace exists, then it will switch to the release branch and run git pull origin (release-branch) to ensure the codebase is
the same as remote.

Then the release process is started with the following step for each repository.

* check out release branch or pull the latest from the release branch.
* generate CHANGELOG.md from local release branch and github.com API.
* check in the CHANGELOG.md to the release branch
* run "mvn clean install deploy -DperformRelease" to release to Maven Central.
* read CHANGELOG.md for the current release into a StringBuffer.
* call GitHub API to create a new release note and tag 

Note that before release, all projects should be built successfully in the release branch by [develop-build][]. After the release, you might need to change the GitHub release a little bit to add the upgrade guide.

For live configuration examples, please visit [light-config-test][].

Here is an example of the release-maven.yml config file for networknt projects in the master branch.

```yaml
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasemaven_1_6_x
# release version that is used to generate changelog. This need to be changed every time
version: 1.6.0
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# previous tag used to calculate how many commits in between in the git log
prev_tag: 1.5.32
# last number of pull requests retrieved from the GitHub. 100 minimum.
last: 100
# You can skip checkout if you are sure that the code in workspace are the latest and
# you just want to repeat the release process due to some environmental issue before.
skip_checkout: false
# skip change log generation
skip_change_log: false
# skip check in the generated changelog
skip_checkin: false
# You can skip the last release step so that you can double check the merged result.
skip_release: false
# skip upload release note to the github
skip_release_note: false
# clone and switch to develop branch / checkout and pull from develop branch
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
    - git@github.com:networknt/light-docker.git
    - git@github.com:lightapi/light-config-server.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-workflow-4j.git
    - git@github.com:networknt/light-spring-boot.git

# generate changelog.md from github issues and check in
# release to maven central
# merge the changelog.md to develop branch
# publish the release to github repository
release:
  - light-4j
  - json-overlay
  - openapi-parser
  - light-rest-4j
  - light-graphql-4j
  - light-hybrid-4j
  - light-codegen
  - light-eventuate-4j
  - light-tram-kafka
  - light-tram-4j
  - light-saga-4j
  - light-session-4j
  - light-spa-4j
  - light-consumer-4j
  - light-router
  - light-proxy
  - light-spring-boot
```

[develop-build]: /tool/light-bot/task/develop-build/
[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/release-maven
