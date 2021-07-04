---
title: "ChangeLog Command"
date: 2019-03-27T09:19:49-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For release-docker and release-maven tasks, we need to generate the changelog for the release and update CHANGELOG.md in the release branch. Previously, we were using [github-changelog-generator][]; however, it doesn't support changelog generation per release branch after we introduced the maintenance branch. 

After searching on the Internet for other alternative tools, we decided to build a command component in the light-bot. Most existing tools don't support release branches and leverage the GitHub v3 Rest API with a lot of extra data fetched from the server. We need release branch support and want to use the V4 GraphQL API to fetch exactly what we need. 

### Commit Logs

The idea is very simple. First, we need to get the commit log within the release branch from the previous tag to the HEAD. This list contains all the changes in the branch since the last release. 

As `git log` only supports DateTime to specify the beginning of the commit log to retrieve. We create one method to get the DateTime of the previous tag with the following git CLI.

```
git log -1 --format=%aI 1.5.32
```

The above command will be executed on the current branch and returned the timestamp of the tag 1.5.33. Here is an example of the returned value.

```
2019-03-19T18:46:58-04:00
```

Now we can use the timestamp to get all the commit log from the timestamp to the latest. 

```
git log --since=2019-03-19T18:46:58-04:00
```

The above command will return the commit logs in the following format per entry.

```
commit 915efccf2cbdaa9805271c14c906c60871234b9a
Author: BalloonWen <whenballoon@gmail.com>
Date:   Wed Mar 20 10:20:20 2019 -0400

    -fixed issue for consul's ttl check, now the ttl interval will be still the same as "checkInterval", but the heartbeat will be 2/3 of "checkInterval" (#428)
```

What we need is to parse each entry to get the pull request number. In the above case, it is 428 at the end of the 5th line. 

After parsing the commit log, we will get a list of pull request numbers that merged into the current release branch since the last release. 

### Pull Requests

With a list of pull request numbers, we need some detailed info for each pull request to generate the formatted changelog for the release. 

We will be using the V4 GitHub API to retrieve the 100 latest pull requests. Here is the GraphQL query.

```
query {
    repository(owner:"networknt", name:"light-4j") {
    pullRequests(last:100, states:MERGED) {
      edges {
        node {
          number,
          title,
          url,
          author {
            login,
            url
          }
        }
      }
    }
  }
}
```

You can run it from the [GitHub explore][] website after logging in with your GitHub account. 

With all the info available, we generate a map from the result of the query. From the pull request list in the first step, we can retrieve the detail from the map and generate the changelog entries for the release. 

### Merge with CHANGELOG.md

Once the entries for the current release are generated, we need to save them into the existing CHANGELOG.md file. This is done by just merging two lists at a certain index. 

### Summary

There are a lot of tools implemented in different languages from GitHub, but none of them are suitable for our needs. With this command, we have full control over the format and logic so that we can customize if necessary. Also, this built-in command will help our users if they are leveraging the light-bot for their DevOps pipeline. 

[github-changelog-generator]: https://github.com/github-changelog-generator/github-changelog-generator
[GitHub explore]: https://developer.github.com/v4/explorer/

