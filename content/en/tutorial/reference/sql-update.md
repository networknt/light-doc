---
title: "SQL Update"
date: 2020-05-03T23:59:48-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Unlike most APIs with NoSQL databases, Reference API has a SQL database for storage. In the light-reference repository, there is ref.sql, which is the source for the production reference. To execute the ref.sql on test4 or prod4, follow the steps below. 

Copy the ref.sql to the remote server


```
cd ~/networknt/light-reference
scp ref.sql prod4:/home/[userid]/ref.sql
```

ssh to the test4 or prod4 and copy the file into the MySQL docker container

```
ssh prod4
docker ps
docker cp ref.sql [container-id]:/ref.sql
```

run the MySQL inside the docker

```
docker exec -it [container-id] mysql -u [mysqluser] -p ref
```

execute the ref.sql

```
source ref.sql
exit

```

restart the reference service

prod4

```
cd ~/networknt/light-config-prod/prod4
docker-compose restart reference
```

test4

```
cd ~/light-chain/light-config-test/test4
docker-compose restart reference

```


To double-check if the dropdown is updated after restarting the container. You can use the following URL to check. Please check the table that has been updated by the script. 


```
https://lightapi.net/r/data?name=covid-category
```

or 

```
https://dev.lightapi.net/r/data?name=covid-category
```





