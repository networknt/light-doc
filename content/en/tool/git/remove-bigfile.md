---
title: "Remove Big File to Reduce Repo Size"
date: 2018-03-09T16:30:44-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true 
---

In the early days, some developers checked in some jar files into several repos in networknt org. The
end result is these repos are very big and it takes a lot of time to clone them. In a process to speed
up our DevOps flow with [light-bot][], it becomes crucial to reduce the size of these repos and keep
the entire history of commits. 

There are several method to do the job but none of them works on its own. Here is the process we followed.


### Consolidate Branches

First step is to merge all branches to master and all branches except master. This makes sure the you
only work on master once and no conflict exists after the process. Somehow, multiple branches cannot
be merged after updating commit log history. 


### Remove all jar files

Once we have consolidated all branches to master, clone it to local and remove the jar files and checkin.

Use this command in the repo directory to find all jar files and remove them. 

```
find . -name "*.jar"
``` 

Check in after all jar files are removed. 

### Find commited jar files in git

There are two tools that can do the job. 

listrepo.sh will list all the file in a repo ordered by file size desc.  

```
#!/bin/bash -e

# work over each commit and append all files in tree to $tempFile
tempFile=$(mktemp)
for commitSHA1 in $(git rev-list --all); do
	git ls-tree -r --long "$commitSHA1" >>"$tempFile"
done

# sort files by SHA1, de-dupe list and finally re-sort by filesize
sort --key 3 "$tempFile" | \
	uniq | \
	sort --key 4 --numeric-sort --reverse

# remove temp file
rm "$tempFile"

```

Another script largefile.sh will list the 10 largest files in a repo.

```
#!/bin/bash
#set -x

# Shows you the largest objects in your repo's pack file.
# Written for osx.
#
# @see https://stubbisms.wordpress.com/2009/07/10/git-script-to-show-largest-pack-objects-and-trim-your-waist-line/
# @author Antony Stubbs

# set the internal field spereator to line break, so that we can iterate easily over the verify-pack output
IFS=$'\n';

# list all objects including their size, sort by size, take top 10
objects=`git verify-pack -v .git/objects/pack/pack-*.idx | grep -v chain | sort -k3nr | head`

echo "All sizes are in kB's. The pack column is the size of the object, compressed, inside the pack file."

output="size,pack,SHA,location"
allObjects=`git rev-list --all --objects`
for y in $objects
do
    # extract the size in bytes
    size=$((`echo $y | cut -f 5 -d ' '`/1024))
    # extract the compressed size in bytes
    compressedSize=$((`echo $y | cut -f 6 -d ' '`/1024))
    # extract the SHA
    sha=`echo $y | cut -f 1 -d ' '`
    # find the objects location in the repository tree
    other=`echo "${allObjects}" | grep $sha`
    #lineBreak=`echo -e "\n"`
    output="${output}\n${size},${compressedSize},${other}"
done

echo -e $output | column -t -s ', '
``` 

### Remove larget files in commits

##### BFG
There is a tool call [BFG repo cleaner][] and it is very convenient if it works. But in practice, it does
work on all use cases. As our repo has a lot of pull requests and we constantly get the following error. 

```
! [remote rejected] refs#1/head -> refs#1/head (deny updating a hidden ref)
``` 

If there is no error in the last step. The following would be the procedure. 

```
git clone --mirror git://example.com/some-big-repo.git
```  
Make a backup and then. 

```
java -jar bfg.jar --strip-blobs-bigger-than 10M some-big-repo.git
```

```
cd some-big-repo.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push
```

##### git filter-branch 

```
git filter-branch --force --tree-filter 'rm -f docker/command/service/eventuate-todo-command-service-0.1.0.jar' HEAD
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force origin master
```

##### git-forget-blob

This is another [tool][] that can remove certain files in the git commit history. 

```
sudo wget https://raw.githubusercontent.com/nachoparker/git-forget-blob/master/git-forget-blob.sh -O /usr/local/bin/git-forget-blob
sudo chmod +x /usr/local/bin/git-forget-blob
```

```
git-forget-blob file-to-forget
```
 
### Check repo size

Once way to check the repo size after push is to clone it on another location and check the size. The
following command can be used to check the size of a directory. 

```
du -sh light-portal
```

### Chrome Extention for github repo size

This tool can be installed from this [link][]. 

[light-bot]: /tool/light-bot/
[BFG repo cleaner]: https://rtyley.github.io/bfg-repo-cleaner/
[tool]: https://ownyourbits.com/2017/01/18/completely-remove-a-file-from-a-git-repository-with-git-forget-blob/
[link]: https://chrome.google.com/webstore/detail/github-repository-size/apnjnioapinblneaedefcnopcjepgkci?hl=en
