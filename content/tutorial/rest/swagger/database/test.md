---
title: "Test"
date: 2017-11-24T08:55:35-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 40	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous steps, we have generated the project and built it. Now the service is up and running. Let's access it from the curl command.

Single query.

```
curl -k https://localhost:8443/v1/query
```

Result:

```json
 {
                                "id": 123,
                                "randomNumber": 456
                            }
```

Multiple queries with the default number of objects returned.

```
curl -k https://localhost:8443/v1/queries
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

Multiple queries with ten objects returned.

```
curl -k https://localhost:8443/v1/queries?queries=10
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

Multiple updates with the default number of objects updated.

```
curl -k https://localhost:8443/v1/updates
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


Multiple updates with ten objects updated.

```
curl -k https://localhost:8443/v1/updates?queries=10
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

As you can see, we have a newly generated project that can output mock data from each endpoint defined in the Swagger 2.0 specification. There is no DB access code behind the scene. In the next step, we are going to create [DB script][] and wire in the code for the MySQL database. 
 

[DB script]: /tutorial/rest/swagger/database/dbscripts/
