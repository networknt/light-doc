---
title: "Portal Host Menu Tutorial"
date: 2018-03-31T09:16:19-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The host-menu service has many endpoints, and here we are going to explore all of them. 

### Create a new host

There are test cases built into the services, and we need to extract these to create curl command line to test the services live. 

Here is the command to create a new host. 

```
curl -k -X POST https://localhost:8443/api/json -H 'content-type: application/json' -d '{"host":"lightapi.net","service":"menu","action":"createMenu","version":"0.1.0","data":{"host":"example.org","description":"example org web site","contains":["1","2","3"]}}'

```

The result should be something like this.

```
{"host":"example.org","description":"example org web site","contains":["1","2","3"]}
```

### Query host menu

Now let's query the host-menu on the query side service to see if the newly created host is shown up. 

```
curl -k -X POST https://localhost:8442/api/json -H 'content-type: application/json' -d '{"host":"lightapi.net","service":"menu","action":"getMenu","version":"0.1.0"}'

```

The result should be something like this.

```
[]
```

