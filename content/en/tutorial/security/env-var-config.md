---
title: "Environment Variable for Config Value"
date: 2020-04-09T11:11:22-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

When working with a light-4j application, there might be a lot of configuration files that need to be checked into the Git repository. For an open-source project, chances are you need to check the source code and configuration files to the public repository on the GitHub. If you have some sensitive information in one or more config files, you need to be really careful not to check the information to the public repository. 

You can use the light-config-server to manage the config values without providing clear text config values in the configuration files; however, it is not very convenient to start the light-config-server locally. Although light-4j config files support encryption and decryption, it is still risky to check the encrypted password, key, etc. to the GitHub as somebody with computing power can brute force the sensitive info. 

The best approach is to check in the config file with a variable template and then put replace it with a system environment variable during the runtime. In this case, we keep the secret locally on our computer and only replace the config value during the runtime. 


In this tutorial, we are going to use the email-sender module to demo the usage.  This module has a config file called email.yml with a password. We don't want to reveal the password to the public GitHub repository, so we need to make the config file to use an environment variable. 


Here is the content of the email.yml file.

```
# Email Sender Configuration
---
# Email server host name or IP address
host: mail.lightapi.net
# Email SMTP port number. Please don't use port 25 as it is not safe
port: 587
# Email user or sender address.
user: noreply@lightapi.net
# Email password
pass: ${NOREPLAY_EMAIL_PASSWORD:password}
# Enable debug. Default to false.
debug: true
# Enable Authentication. Default to true.
auth: true

```

The pass default value is `password` but it can be replaced with an environment variable named `NOREPLAY_EMAIL_PASSWORD`. 

To verify if it works, we need to create a test case to ensure it is working. The test case looks like the following. 

```
@Test
public void testConfigPassword() {
    EmailConfig config = (EmailConfig)Config.getInstance().getJsonObjectConfig(EmailConfig.CONFIG_NAME, EmailConfig.class);
    System.out.println("password = " + config.pass);
}
```

It loads the config file and outputs the pass to the terminal. Now, let's run the test case from a terminal. 

```
cd ~/networknt/light-4j/email-sender
mvn test
```

You will see the following line in the output. 

```
password = password
```

The default value is used as we don't have the environment variable defined on my desktop yet.

Let's define the environment variable in the command line. 

```
export NOREPLAY_EMAIL_PASSWORD=abc
```

Rerun the `mvn test`, and you can see the `password = abc` in the output.

To make it persistent, you can put the export into your .profile file in your home directory. After that, your application can only send out emails on your computer. 

For the source code mentioned in this tutorial, please take a look at this commit. 

https://github.com/networknt/light-4j/issues/690
