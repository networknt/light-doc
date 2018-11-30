---
title: "IntelliJ Idea"
date: 2018-11-24T13:25:31-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Most of our developers are using IntelliJ Idea as the IDE when working on light-platform or building application on the light platform. Its community edition is free and has all the features we need.

### Settings

There are several personal settings that might be useful for others to increase productivity. 

##### Change font size

I am using a 40 inch 4K monitor and have more screen real estate. It would be much easier to increase the font size when needed. To do that, got to File/Settings/Editor/General. In the Mouse section, check the "Change font size(Zoom) with Ctrl+Mouse Wheel"


##### Inotify Watches Limit

IntelliJ IDEA is monitor the filesystem changes and reload the files into the IDE once files are changed from outside. If you have bigger project with thousands of files, you need to increate the watch limit. 

- Add the following line to either /etc/sysctl.conf file or a new *.conf file (e.g. idea.conf) under /etc/sysctl.d/ directory:


```
fs.inotify.max_user_watches = 524288
```

- Run this command to apply the change:


```
sudo sysctl -p --system
```

Restart your IDE once it is done. 


**Note:** the watches limit is per-account setting. If there are other programs running under the same account which also uses Inotify the limit should be raised high enough to suit needs of all of them.
