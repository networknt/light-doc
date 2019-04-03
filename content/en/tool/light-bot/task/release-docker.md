---
title: "Release Docker"
date: 2019-03-27T14:04:58-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This is the task to release all services for the networknt organization to the docker hub. It first checkout repositories into a workspace and switch to the release branch defined in the release-docker.yml file. If the workspace exists, then it will switch to the release branch and run git pull origin (release-branch) to ensure the codebase is the same as remote.

Then the release process is started with the following step for each repository.

* check out release branch or pull the latest from the release branch.
* generate CHANGELOG.md from local release branch and github.com API.
* check in the CHANGELOG.md to the release branch
* run `mvn clean install` to build the service
* build docker image with the Dockerfile in each service
* push the docker images to the docker hub
* read CHANGELOG.md for the current release into a StringBuffer.
* call GitHub API to create a new release note and tag 

Note that before release, all projects should be built successfully in the release branch by [develop-build][]. After the release, you might need to change the GitHub release a little bit to add the upgrade guide.

For live configuration examples, please visit [light-config-test][].

Here is an example of the release-docker.yml config file for networknt projects in the master branch.

```
# Workspace that is used for this operation. Most of time, this is done on local.
workspace: releasedocker_1_6_x
# new upgraded version. This needs to be changed for every release.
version: 1.6.0
# github organization as the token is bound to the org in changelog generator. This
# means that you can only release multiple repositories within the same org each time.
organization: networknt
# previous tag used to calculate how many commits in between in the git log
prev_tag: 1.5.32
# last number of pull requests retrieved from the GitHub. 100 minimum.
last: 100
# skip checkout if you know all latest code has been checkout in the previous build
skip_checkout: false
# skip maven build if you know the build has been done already
skip_maven: false
# skip generate change log
skip_change_log: false
# skip check in the generated changelog
skip_checkin: false
# skip upload release_note if it is already done.
skip_release_note: false
# skip docker publish if it is done already.
skip_docker: false
# clone and switch to develop branch / checkout and pull from develop branch
checkout:
  # here we checkout master branch to publish the images. 
  - branch: master
    repository:
    - git@github.com:networknt/light-proxy.git
    - git@github.com:networknt/light-router.git
    - git@github.com:networknt/light-oauth2.git
    - git@github.com:networknt/light-tokenization.git
    - git@github.com:networknt/light-portal.git
    - git@github.com:networknt/light-eventuate-4j.git
    - git@github.com:networknt/light-tram-kafka.git
    - git@github.com:networknt/light-tram-4j.git
    - git@github.com:networknt/light-example-4j.git
    # - git@github.com:networknt/light-codegen.git
    # - git@github.com:networknt/light-docker.git
# maven build
maven:
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-portal
  - light-eventuate-4j
  - light-tram-kafka
  - light-tram-4j
  - light-example-4j/discovery/api_a/kubernetes
  - light-example-4j/discovery/api_b/kubernetes
  - light-example-4j/discovery/api_c/kubernetes
  - light-example-4j/discovery/api_d/kubernetes
  # - light-codegen

# update changelog in master, merge it to develop and publish to github.com
# please note that light-eventuate-4j and light-tram-4j are released in release-maven task.
release:
  - light-proxy
  - light-router
  - light-oauth2
  - light-tokenization
  - light-portal
  # - light-codegen

# build docker image and push to docker hub
docker:
  - light-proxy
  - light-router
  - light-oauth2/authorize
  - light-oauth2/client
  - light-oauth2/code
  - light-oauth2/key
  - light-oauth2/refresh-token
  - light-oauth2/service
  - light-oauth2/token
  - light-oauth2/user
  - light-tokenization
  - light-portal/hybrid-command
  - light-portal/hybrid-query
  - light-example-4j/discovery/api_a/kubernetes
  - light-example-4j/discovery/api_b/kubernetes
  - light-example-4j/discovery/api_c/kubernetes
  - light-example-4j/discovery/api_d/kubernetes
  # need to shutdown docker-compose-integration-test.yml before building the following two images.
  # moved to release-cdc sub task.
  # - light-eventuate-4j/eventuate-cdc-server
  # - light-tram-4j/tram-cdc-mysql-server
  # - light-docker/light-codegen
```

[light-config-test]: https://github.com/networknt/light-config-test/tree/master/light-bot/release-docker
