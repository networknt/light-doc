---
title: "ssh"
date: 2018-02-19T10:51:16-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

To make the connection to the remote server easier, let's first add an entry to /etc/hosts for the `devops` server so that we don't need to remember the IP address all the time.

```
127.0.0.1       localhost
192.168.1.120   joy
xx.xxx.xxx.xx   devops
``` 

Now, if you want to connect to the `devops` server from your desktop, you can just issue

```
ssh devops
```

You can see that I don't have any user before `devops` because my current user on the desktop is the same on the `devops` server. Otherwise, you have to use a command like this.

```
ssh user@devops
``` 

Now you can connect to the `devops` server with ssh after you provide your password. It is still not very convenient as you have to type your password every time.

The next step, we can upload the public key to the server so that we don't need to type the password anymore. 

For the steps to generate the key and upload it, please refer to

https://www.ssh.com/ssh/copy-id

The following are my commands for reference only.

```
ssh-copy-id -i ~/.ssh/id_rsa.pub steve@devops
```  

From now on, you can type `ssh devops` to log in to the server without providing a password. It is an essential step to allow light-bot to run commands on the remote server automatically if you want to leverage it. 
