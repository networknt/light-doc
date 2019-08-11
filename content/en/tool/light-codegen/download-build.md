---
title: "Download vs Build light-codegen"
date: 2019-08-10T18:07:05-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Most users are using the Java command line to invoke light-codegen to generate projects locally. You can download the codegen-cli.jar from the release page or clone the light-codegen repository and build locally. 

The easiest way is to download the codegen-cli.jar from the release page. For each release, we upload the fat jar to the release page as an asset for users to download. Once the file is downloaded, put into a working directory like `~/tool` or `~/networknt` and run the command line from the same folder. 

If you want to customize or debug the light-codegen, you can clone the light-codegen GitHub repository and build it locally. You can also run it within the IDE with debugging is turned on. Please note that the default branch is `release` for light-codegen, which is the last released version of the main development branch master. If you want to generate a project for a particular release, please use `git checkout origin release-tag` to check out the specific tag. For example, `git checkout origin 1.6.6`. 

Here is one example to check out the light-codegen into the workspace networknt under your home directory. 

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
# if you don't want to use the latest release of the master branch
# git checkout origin 1.6.6
cd light-codegen
mvn clean install
```

You can find all the release tags and jars from the release page at https://github.com/networknt/light-codegen/releases.
