---
title: "Test"
date: 2017-11-24T08:55:35-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Now the service is up and running. Let's access it from curl

Single query

```
curl http://localhost:8080/v1/query
```

Result:

```json
 {
                                "id": 123,
                                "randomNumber": 456
                            }
```

Multiple queries with default number of object returned

```
curl http://localhost:8080/v1/queries
```

Result:

```json
 [
                                {
                                    "id": 123,
                                    "randomNumber": 456
                                },
                                {
                                    "id": 567,
                                    "randomNumber": 789
                                }
                            ]
```

Multiple queries with 10 numbers returned

```
curl http://localhost:8080/v1/queries?queries=10
```

Result: 

```json
 [
                                {
                                    "id": 123,
                                    "randomNumber": 456
                                },
                                {
                                    "id": 567,
                                    "randomNumber": 789
                                }
                            ]
```

Multiple updates with default number of object updated

```
curl http://localhost:8080/v1/updates
```

Result:

```json
 [
                                {
                                    "id": 123,
                                    "randomNumber": 456
                                },
                                {
                                    "id": 567,
                                    "randomNumber": 789
                                }
                            ]
```


Multiple updates with 10 numbers updated

```
curl http://localhost:8080/v1/updates?queries=10
```

Result:

```json
 [
                                {
                                    "id": 123,
                                    "randomNumber": 456
                                },
                                {
                                    "id": 567,
                                    "randomNumber": 789
                                }
                            ]
```
