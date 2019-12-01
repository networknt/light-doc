---
title: Hugo Docs
date: 2017-11-04T22:12:19-04:00
description: "Hugo for project documentation on github"
categories: []
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

For most open source developers, they would use github.com to manage their projects and then have a README.md for documentation. For some bigger projects, they might have a wiki for additional documents and linked them from README.md.

**This tutorial is written for Linux and Mac users; however, it should not be hard to follow by Windows users.**

Here I will introduce another way to manage documents in gh-pages with [Hugo](https://gohugo.io/) static site generator. It is suitable for small projects with minimum documentation. If you are working on a more significant project, please create a [Hugo site][] instead. 


## Installation


### Hugo


- Manual

For Linux, download the binary from the [latest release](https://github.com/gohugoio/hugo/releases)

I am using hugo_0.51_Linux-64bit.deb on Ubuntu and install it with the following command.


```
sudo dpkg -i hugo_0.51_Linux-64bit.deb
```

To verify that Hugo is installed successfully.

```
hugo version
```

- Package Manager

Another way to install Hugo is to use the package manager on your Linxu distro. For example, if you are using Ubuntu 18.04, then you can install HUgo by issuing the following command. This is not recommended at the moment as most Linux distros have only older version of the Hugo available. 

```
sudo apt install hugo
```

For Mac, the easiest way is to use Homebrew.

```
brew update && brew install hugo
```


### Pygments

The Hugo executable has one optional external dependency for source code highlighting (Pygments). To install it.

```
pip install Pygments
```

### Enable Hugo Bash Auto-Completion

```
sudo hugo gen autocomplete
. /etc/bash_completion
```

Now if you type `hugo` followed by a few TABs, you will see the commands that Hugo recommended.

## Create gh-pages branch

GitHub Pages will serve up a website for any repository that has a branch called `gh-pages` with a valid index.html file at that branch’s root.

1. Create an orphaned `gh-pages` branch

To set up a Project Pages site, you need to create a new "orphan" branch (a branch that has no common history with an existing branch) in your repository. The safest way to do this is to start with a fresh clone:

```
git clone github.com/user/repository.git
cd repository
git checkout --orphan gh-pages
```

2. Remove all files to create an empty working directory

```
git rm -rf .

```

3. Add test index.html and push

```
echo "My Page" > index.html
git add index.html
git commit -a -m "initial checkin"
git push origin gh-pages
```

4. Test your site

After the push to `gh-pages` branch, your site will be available at the following URL.

```
https://<orgname>.github.io/<reponame>
```

## Scaffold docs site

First switch to master branch

```
git checkout master
```

Create a new docs site

```
hugo new site docs
```

Now you have a docs directory in your project root folder and some empty subdirectories under docs. By default, git will not commit empty directories to the repository. To work around it, we can include .gitkeep file in each of these directories.

```
cd docs
for DIR in `ls -p | grep /`; do touch ${DIR}.gitkeep; done
```

To verify that these files are created.

```
find . -name .gitkeep
```
## Config

Copy the content of [my config](https://raw.githubusercontent.com/networknt/light-4j/master/docs/config.toml) into config.toml under docs and update it accordingly

## Theme

I am using `hugo-material-docs` theme and here are the steps to install it.

```
cd docs/themes
git clone git@github.com:digitalcraftsman/hugo-material-docs.git
```

## Content

Now let's create the first page and test it.

1. From docs folder, run

```
hugo new index.md
```

2. Find the file docs/content/index.md and update it with some testing content.

3. Start the server and test it.

From docs folder

```
hugo server
```

4. Now you about.md can be accessed at

```
http://localhost:1313/<reponame>
```

## Deploy

Get the deploy.sh

```
# Fetch the deployment script into the root of your source tree, make it executable.
wget https://github.com/X1011/git-directory-deploy/raw/master/deploy.sh && chmod +x deploy.sh

```

From docs folder run the following command to publish your site to `gh-pages`. This can be repeated every time you want to publish your site.

```
# For setting it up to build to a folder other than "dist", see the options in deploy.sh.
# Build the site to /dist.
hugo -d dist

# Run the deploy.sh script installed above.
./deploy.sh
```

Now your document site can be accessed from https://<username>.github.io/<projectname>

## Check in

Your site is up and running now, let's check it in before adding more documents.

```
cd ..
git add .
git commit -m "add docs for documentation"
git push origin master
```

[Hugo site]: /tool/hugo-site/

