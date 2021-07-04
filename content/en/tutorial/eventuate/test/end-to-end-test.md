---
title: "End to End Test"
date: 2018-01-06T20:30:04-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

End-to-End test in light-eventuate-4j application is very hard as many components need to
work together in order for message to be delivered from one service to another. Also, each
service instance must live long enough in order to receive async events. Internally, we
have so many services that are built on top of light-eventuate-4j and light-hybrid-4j and we
are using [light-bot][] develop task to test them all during the develop branch build. 

For more information, please refer to generic [end-to-end test tutorial][]


### End to End Test

As an example, we have integrated end-to-end tests for [todo-list][] into the default develop
build in light-bot. 

Here is the section that is for end-to-end test for todo-list.

```
TBD
```


[light-bot]: https://github.com/networknt/light-bot
[end-to-end test tutorial]: /tutorial/common/test/end-to-end-test/
[todo-list]: /tutorial/eventuate/todo-list/
