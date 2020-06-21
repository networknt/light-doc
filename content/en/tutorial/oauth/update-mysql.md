---
title: "Update MySQL Docker"
date: 2020-06-21T11:00:34-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

The deployed light-oauth2 services on the test cloud and prod cloud are using the MySQL database. To support multi-tenancy, we need to add a host column to the client and service table. The database scripts are updated in all repositories. And we need to update the live database without recreating it on test4 and prod4 server. 

The following are steps performed on the test4 server. 

```
ssh test4
docker ps
docker exec -it [container-id] mysql -u [mysqluser] -p oauth2
ALTER TABLE client ADD host VARCHAR(32) NOT NULL DEFAULT 'lightapi.net';
ALTER TABLE service ADD host VARCHAR(32) NOT NULL DEFAULT 'lightapi.net';
exit
```

Perform the same steps on prod4 to update the production MySQL database. 
