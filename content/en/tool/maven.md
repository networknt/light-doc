---
title: "Maven"
date: 2018-02-19T11:09:05-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to show you how to install Maven on a Ubuntu server. 

First you need to go to apache.org to find out the url to the latest Maven bin.tar.gz

The current version is 

http://apache.mirror.rafal.ca/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz 

```
cd /usr/local
sudo wget http://apache.mirror.rafal.ca/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
sudo tar xvf apache-maven-3.5.2-bin.tar.gz

```

Now, let's update .bashrc file under your home directory to export environment variables.

```
export M2_HOME=/usr/local/apache-maven-3.5.2
export M2=$M2_HOME/bin
export PATH=$M2:$PATH
```

Save the file and run

```
source .bashrc
```

To verify if Maven is installed successfully. 

```
mvn -version
```
