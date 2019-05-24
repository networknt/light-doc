---
title: "Light-bot Email Configuration"
date: 2019-03-27T14:53:41-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Some users are using the light-bot tasks on their local computer for daily job. Other users might install the light-bot on their remote server to run some tasks triggered by Cron jobs. If you run tasks remotely several times per day, an email notification is crucial. 

To enable email notification, you need to make the following configuration changes. 

* cli.yml

You need to set skipEmail to false to enable email sending.

```
# This is the config file for Cli command line
# Indicate if there is an email will be sent out if the build fails.
skipEmail: true
```

* email.yml

You need to have an email configuration file to specify email server and login info.

```
# Email Sender Configuration
---
# Email server hostname or IP address
host: mail.lightapi.net
# Email SMTP port number. Please don't use port 25 as it is not safe
port: 587
# Email user or sender address.
user: noreply@lightapi.net
# Enable debug. Default to false.
debug: true
# Enable Authentication. Default to true.
auth: true
```

* secret.yml

If you want to use encrypted email password, you can use the secret.yml or the email.yml 

The other option is to set the email password as an environment variable in .bashrc or .profile
