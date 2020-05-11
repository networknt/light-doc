---
title: "Hugo Netlify"
date: 2018-10-28T21:37:41-04:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

Hugo is a static site generator and the generated site can be deployed to gh-pages branch as [hugo-docs][] branch of your repository or a [hugo site][]. If you want to make the entire static site automatic, you can leverage a great service provided by netlify.com

For how to install Hugo and work with the command line tool, please refer to the reference document at the end. 

### Create a repo

Now let's create a repo for the static site. Let's say we are using https://github.com/networknt/doc.maproot.net repository. For easy start, I am copying from the latest https://github.com/gohugoio/hugoDocs as the starting point. 

When copying the site, please don't copy the themes but just copy the following folders. 

```
drwxrwxr-x 2 steve steve 4096 May 11 12:45 archetypes
drwxr-xr-x 5 steve steve 4096 Aug 17  2019 config
-rw-rw-r-- 1 steve steve 1717 May 11 12:24 config.toml
drwxrwxr-x 4 steve steve 4096 May 11 11:24 content
drwxr-xr-x 2 steve steve 4096 May 11 11:09 data
drwxr-xr-x 5 steve steve 4096 Aug 17  2019 layouts
-rw-rw-r-- 1 steve steve 1066 May 11 11:00 LICENSE
-rw-rw-r-- 1 steve steve  660 May 11 11:09 netlify.toml
-rwxr-xr-x 1 steve steve  120 Aug 17  2019 pull-theme.sh
-rw-rw-r-- 1 steve steve 1043 May 11 11:37 README.md
drwxrwxr-x 3 steve steve 4096 May 11 12:58 resources
drwxrwxr-x 4 steve steve 4096 May 11 11:26 static
drwxrwxr-x 3 steve steve 4096 May 11 12:21 themes

```

Checkin the newly copied site and then let's add the theme. 

```
git subtree add --prefix=themes/gohugoioTheme/ git@github.com:gohugoio/gohugoioTheme.git master --squash
```

After that, make the change in the themes folder to customize the template. Once finished, check in the result. The update for the theme templates are checked in as well so that we can keep the updates. 


The following is a document to help to setup the netlify. 

https://gohugo.io/hosting-and-deployment/hosting-on-netlify/

Remember that it will take up to 24 hours for the DNS to propagate once you update the cloudflare DNS. Once the DNS record is propagated, you can enable TLS with netlify certificate.



[hugo-docs]: /tool/hugo-docs/
[hugo site]: /tool/hugo-site/
