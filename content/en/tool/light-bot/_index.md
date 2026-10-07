---
title: "Light Bot"
date: 2017-12-30T07:15:31-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

One of the design principles of `light-4j` frameworks is to make each Lego piece smaller so that developers have the flexibility to compose their services with just enough functionality and smaller memory footprint/smaller delivery package. As you can see, most light-4j cross-cutting modules only have about five Java classes on average, and we have a lot of modules to manage.

It is even more complicated since we support different [style of interaction][] for microservices, and we have to create several frameworks as different customers are using different frameworks.

It has a significant negative impact on package management, testing and release process as there are many more moving pieces. Every time we have a release of frameworks, there are at least two days in testing to make sure that everything works together. 

We have been looking at DevOps tools on the open-source world to streamline our process, but all of them are not designed for us. They are focusing on a single repository monolithic app only as most of them build a single project by putting a config file in the repository. Before we decide to build our own light-bot, we have spent countless hours on Jenkins and try to make it work for our [repositories][], and one customer who has dozens of microservices finally gave up. Here is a list of [Jenkins issues][], and I think other DevOps tools have these issues in common. 

We need an automation tool that can handle multiple repositories with dependencies for our packages, and at the same time, our customers are asking for a similar tool to manage their microservices once the numbers are increasing. 

This forces us to consider building our own DevOps tool [light-bot][]  

To start a full-blown DevOps toolchain is not an easy job, and we are starting from the command line first and will add a central control server and agent server later on. 

light-bot uses Maven for its modules, tests, and executable CLI packaging.
Building requires JDK 25 or newer and Maven 3.6.3 or newer. Running the CLI
also requires Java 25 or newer.

To build the project, run `mvn clean verify` in the root folder of [light-bot][].
The executable JAR is `bot-cli/target/bot-cli.jar`. Supply complete external
task configuration with `-Dlight-4j-config-dir=/path/to/config` before `-jar`.

- [Why light-bot][]
- [NetworkNT repositories][]
- [How light-bot works][]
- [Build light-bot][]
- [NetworkNT configurations][]
- [Invoke bot-cli][]
- [Command](/tool/light-bot/command/)
  * [ChangeLogCmd](/tool/light-bot/command/change-log/)
- [Task](/tool/light-bot/task/)
  * [Develop Build](/tool/light-bot/task/develop-build/)
  * [Regex Replace](/tool/light-bot/task/regex-replace/)
  * [Create Branch](/tool/light-bot/task/create-branch/)
  * [Merge Branch](/tool/light-bot/task/merge-branch/)
  * [Release Docker](/tool/light-bot/task/release-docker/)
  * [Release Maven](/tool/light-bot/task/release-maven/)
  * [Release Snapshot](/tool/light-bot/task/release-snapshot/)
  * [Version Upgrade](/tool/light-bot/task/version-upgrade/)
- [Email](/tool/light-bot/email/)

[style of interaction]: /style/
[light-bot]: https://github.com/networknt/light-bot
[Kotlin DSL]: https://github.com/gradle/kotlin-dsl
[light-bot tutorial]: /tutorial/bot/
[networknt/github-changelog-generator]: https://github.com/skywinder/github-changelog-generator
[Jenkins issues]: /tool/light-bot/jenkins/
[Why light-bot]: /tool/light-bot/why-light-bot/
[repositories]: /tool/light-bot/respository/
[NetworkNT repositories]: /tool/light-bot/repository/
[Build light-bot]: /tutorial/bot/build-light-bot/
[How light-bot works]: /tool/light-bot/how-it-works/
[NetworkNT configurations]: /tool/light-bot/networknt-config/
[Invoke bot-cli]: /tool/light-bot/bot-cli/
