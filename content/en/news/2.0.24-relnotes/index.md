---
title: "2.0.24 Light Platform Released"
date: 2021-03-06
description: "A release with several defect fixes and enhancements"
categories: ["Releases"]
toc: true
---

## [2.0.24](https://github.com/networknt/light-4j/tree/2.0.24) (2021-02-24)

**Merged pull requests:**


- fixes \#920 update CookiesDumper in DumpHandler after upgrade to under… [\#921](https://github.com/networknt/light-4j/pull/921) ([stevehu](https://github.com/stevehu))
- Bump version.jackson from 2.10.4 to 2.12.1 [\#919](https://github.com/networknt/light-4j/pull/919) ([dependabot](https://github.com/apps/dependabot))
- issue \#897 key resolving at the start up [\#918](https://github.com/networknt/light-4j/pull/918) ([BalloonWen](https://github.com/BalloonWen))
- fixes \#916 register the handler and server modules to server info [\#917](https://github.com/networknt/light-4j/pull/917) ([stevehu](https://github.com/stevehu))
- fixes \#914 move the getFileExtension from light-codegen to the NioUti… [\#915](https://github.com/networknt/light-4j/pull/915) ([stevehu](https://github.com/stevehu))
- allow key injection in configuration [\#913](https://github.com/networknt/light-4j/pull/913) ([BalloonWen](https://github.com/BalloonWen))
- issue \#898 log err when get oauth key exception [\#911](https://github.com/networknt/light-4j/pull/911) ([BalloonWen](https://github.com/BalloonWen))
- fixes \#909 make shutdown timeout and shutdown graceful period configu… [\#910](https://github.com/networknt/light-4j/pull/910) ([stevehu](https://github.com/stevehu))
- fixes \#906 remove primary and secondary jks from security resources/c… [\#907](https://github.com/networknt/light-4j/pull/907) ([stevehu](https://github.com/stevehu))


**Upgrade Guidelines:**

This is a release with some bug fixes and enhancements. It is backward compatible with the 2.0.23 release. Along with the PRs above, we have upgraded Undertow to 2.2.4.Final and json-schema-validator to 1.0.49.

For all the changes for the entire platform, please refer to https://trello.com/b/189msq9S/release-schedule

