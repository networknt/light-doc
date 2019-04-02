---
title: "Development"
date: 2018-04-22T15:04:32-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "contribute"
    weight: 10
weight: 10
sections_weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

The light platform consists dozens of open-source projects and lives by the work of its contributors. There are plenty of open issues and [request for comments][rfcs], and we need your help to make Light even brighter. You do not need to be a Java/Kotlin/Javascript guru to contribute to the projects.

### Contribution Flow

* For issues or defects, please follow the [issues-defects][] process to create GitHub issues

* For new features, please follow the [new-features][] process to create GitHub issues

* For every change, we need to ensure that the corresponding document is updated in the [light-doc](https://github.com/networknt/light-doc) repository to reflect the change. The reviewers will enforce this rule during the review. 

* Any functional change needs one or more test cases to ensure the quality of the code and help the reviewer to understand how the feature works. The reviewers will enforce this rule during the review. 

* Any new source file that is written by the contributor should have the license/copyright header copied from another file in the same repository. The contributor should also put his/her name as the author of the code. 

* If the entire file or parts of the file is copied from another open source repository, please put their license/copyright in the header of the file and leave the author intact. You can put your name as part of the author if you modify the source code. 

* As version 1.5.x is used on production, all changes should be bug fixes and need to be backward compatible. The reviewers need to enforce this rule. For broken changes, please merge to master branch which is the base for the 1.6.x release. 

* For all the repositories that are used on productions or used by a lot of projects, the master, jkd11 and 1.5.x branches are locked down. Only the organization admin and repository owner can merge the PR to these branches. 

* For any substantial change, it needs to be reviewed by at least two reviewers. For low-risk changes, show stoppers and urgent defect fixes, the organization admin or repository owner can overwrite the rule to review and merge directly. In most of the cases, the person who opens the issue will be added to the reviewer list to ensure that the requirement is met.  

* For corporate contributions, one or two team leads must review and approve a PR before the PR can be reviewed and approved by the community member. 

* Every team is encouraged to submit integration test cases to light-bot [build-test-all](https://github.com/networknt/light-config-test/tree/master/light-bot/develop-build) to cover their application to ensure new changes won't break their applications.

* To avoid duplicate work, each developer should self-signed up to three issues to him/her to indicate that the issues are in progress. People shouldn't work on issues already signed to other people; however, you can politely ask the assignee to reassign an issue to you if you see there is no activity for the issue over a period of time. 

* The process is quite restricted, and it might hinder productivity. The organization admin and repository owner have the ultimate right to overwrite the rules and speed up the process. In the future, we are going to assign one or two owners per repository. 

* If you have a question about the contribution process, please visit [develop guide][] or ask in the [gitter](https://gitter.im/networknt/light-4j) room. 


[rfcs]: https://github.com/networknt/light-rfcs
[docscontrib]: /contribute/documentation/
[light-bot tutorial]: /tutorial/bot/local-develop/
[light-bot]: /tool/light-bot/
[develop guide]: /contribute/developer-guide/
[issues-defects]: /contribute/issues-defects/
[new-features]: /contribute/new-features/
