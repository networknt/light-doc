---
title: "Hugo Netlify"
date: 2018-10-28T21:37:41-04:00
description: ""
categories: []
keywords: []
menu:
  docs:
    parent: "tool"
    weight: 11
weight: 11	#rem
aliases: []
toc: false
draft: false
reviewed: true
---

Hugo is a static site generator and the generated site can be deployed to gh-pages branch as [hugo-docs][] branch of your repository or a [hugo site][]. If you want to make the entire static site automatic, you can leverage a great service provided by netlify.com

For how to install Hugo and work with the command line tool, please refer to the reference document at the end. 

### Create a repo

Now let's create a repo for the static site. Let's say we are using https://github.com/taiji-chain/taiji.io repository. For easy start, I am copying https://github.com/networknt/light-doc as the starting point. 

When copying the site, please don't copy the themes but just create the folder. 

The real theme will be checked out from the github.com manually. 

```
cd taiji.io/themes
git clone 
```



[hugo-docs]: /tool/hugo-docs/
[hugo site]: /tool/hugo-site/
