---
title: "Developer Guide"
date: 2019-02-02T13:53:06-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

The light platform consists dozens of open-source projects and lives by the work of its contributors. There are plenty of open issues and [request for comments][], and we need your help to make Light even brighter. You do not need to be a Java/Kotlin/Javascript guru to contribute to the projects.

### Assumptions

This contribution guide takes a step-by-step approach in hopes of helping newcomers. Therefore, we only assume the following:

* You are new to Git or open-source projects in general
* You are a fan of the light platform and enthusiastic about contributing to the projects

{{% note "Additional Questions?" %}}
If you are struggling at any point in this contribution guide, reach out to the community in [gitter](https://gitter.im/networknt/light-4j) or [reddit](https://www.reddit.com/r/lightapi/) or [Google Group](https://groups.google.com/forum/#!forum/light-4j).
{{% /note %}}

### Install Java

The installation of Java should take only a few minutes. You have more than one option to get Java up and running on your machine. The current master branch is built for JDK 8, and a separate jdk11 branch is built for JDK 11. 

https://www.java.com/en/download/help/download_options.xml

### Install Maven

Maven is our building tool, and it must be installed to compile the light projects. 

https://maven.apache.org/install.html


### Create a GitHub Account

If you are going to contribute code, you need to have an account on GitHub. Go to [www.github.com/join](https://github.com/join) and set up a personal account.

### Install Git

You need to have Git installed on your computer to contribute to Light development. Teaching git is outside the scope of this document, but if you are looking for an excellent reference to learn the basics of Git, we recommend the [Git book][gitbook] if you are not sure where to begin. 

Git is a [version control system](https://en.wikipedia.org/wiki/Version_control) to track the changes of source code. 

Move back to the terminal and check if Git is already installed. Type in `git version` and press enter. You can skip the rest of this section if the command returned a version number. Otherwise [download](https://git-scm.com/downloads) the latest version of Git and follow this [installation guide](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

Finally, check again with `git version` if Git was installed successfully.

### Set up workspace

The working copy is set up locally on your computer in a workspace. It's what you'll edit, compile, and end up pushing back to GitHub. The main steps are forking a repository and cloning the repository to your workspace. To make it simpler, we recommend using networknt under your user home directory as your workspace. 

### Fork the repository

If you are not familiar with this term, GitHub's [help pages](https://help.github.com/articles/fork-a-repo/) provide a simple explanation:

> A fork is a copy of a repository. Forking a repository allows you to freely experiment with changes without affecting the original project.

Open a GitHub repository and click on the "Fork" button in the top right.

Now open your fork repository on GitHub and copy the remote url of your fork. You can choose between HTTPS and SSH as the protocol that Git should use for the following operations. HTTPS always works [if you are not sure](https://help.github.com/articles/which-remote-url-should-i-use/).

### Clone the repository

We assume that you have set up your local development environment (see the section above if you are unsure about this). You should now copy one of the repositories you want to contribute down to your computer. You will hear this called "clone the repo". GitHub's [help pages](https://help.github.com/articles/cloning-a-repository/) give us a short explanation:

> When you create a repository on GitHub, it exists as a remote repository. You can create a local clone of your repository on your computer and sync between the two locations.

We are going to clone the [light-4j repository](https://github.com/networknt/light-4j) as an example. As you have forked the repository in the previous step, you should clone your forked repository instead. 

```
git clone https://github.com/networknt/light-4j.git
```
Or 
```
git clone git@github.com:networknt/light-4j.git
```

### Contribute to master

For each repository, the master branch is the main branch for contribution. The development team will not accept a pull request against other release branches. If you have permission to access networknt repositories, create a descriptive named branch from master branch and work on it. Alternatively, work directly on master branch in your forked repository. 

First, you should always pull the latest changes from the master branch:

```
git checkout master
git pull
```

Now we can create a new branch for your additions:

```
git checkout -b <BRANCH-NAME>
```

You can check on which branch you are with `git branch`. You should see a list of all local branches. The current branch is indicated with a little asterisk.

### Build Project

While making changes in the codebase, it is a good idea to build the binary to test them:

```
mvn clean package
```

When you are working on the master branch, chances are there are some dependencies are missing if you are not working on the light-4j project. Here is a [light-bot tutorial][] to help you to build all repositories in one shot. 


### Test Project

Sometimes, changes in the codebase can cause unintended side effects. Or they do not work as expected. Most functions have their own test cases. You can find them in the test or the integration folder along with the main folder under src.

Make sure the commands `mvn clean package` passes, and `mvn clean install` passes before you commit. 

### Create an Issue

If there is no issue available for what you are planning to work on, please create an issue in the corresponding repository to track what you are about to change. When checking in your code, you can specify the issue number in the comment so that the change log will be linked to the GitHub issue. 

```
git commit -m "fixes #10 add a new test case"
```

### Modify commits

You noticed some commit messages do not fulfill the code contribution guidelines or you forget something to add some files? No problem. Git provides the necessary tools to fix such problems. The next two methods cover all common cases.

If you are unsure what a command does leave the commit as it is. We can fix your commits later in the pull request.

##### Modify the last commit

Let's say you want to modify the last commit message. Run the following command and replace the current message:

```
git commit --amend -m "YOUR NEW COMMIT MESSAGE"
```

Take a look at the commit log to see the change:

```
git log
# Exit with q
```

After making the last commit, you may forget something. There is no need to create a new commit. Just add the latest changes and merge them into the intended commit:

```
git add --all
git commit --amend
```

### Push commits

To push our commits to the fork on GitHub we need to specify a destination. A destination is defined by the remote and a branch name. When you fork to your personal account, the remote url of our fork is the same as our GitHub handle, in my case `stevehu`. The branch should have the same as our local one. This makes it easy to identify corresponding branches.

```
git push --set-upstream <YOUR-GITHUB-USERNAME> <BRANCHNAME>
```

Now Git knows the destination. Next time when you push commits you just need to enter `git push`.

If you modified your commit history in the last step, GitHub will reject if you try to push. This is a safety feature because the commit history is not the same and new commits cannot be appended as usual. You can enforce this push explicitly with `git push --force`.

### Open a pull request

We made a lot of progress. Good work. In this step, we finally open a pull request to submit our additions. Open the repository on GitHub in your browser.

You should find a green button labeled with "New pull request". But GitHub is clever and probably suggests you a pull request like in the beige box below:

![Open a pull request](/images/contribute/development/open-pull-request.png)

The new page summaries the most important information of your pull request. Scroll down, and you find the additions of all your commits. Make sure everything looks as expected and click on "Create pull request".

<!--
### Accept the contributor license agreement

Last but not least you should accept the contributor license agreement (CLA). A new comment should be added automatically to your pull request. Click on the yellow badge, accept the agreement and authenticate yourself with your GitHub account. It just takes a few clicks and only needs to be done once.

![Accept the CLA](/images/contribute/development/accept-cla.png)
-->

### Automatic builds

We use the [light-bot][] (Linux and OS&nbsp;X) to compile light-platform projects together with your additions. It should ensure that everything works as expected before merging your pull request. 

If you have questions, leave a comment on the pull request. We are willing to assist you.


### Contribute to Documentation

For core developers who have write permission to networknt repositories are required to update the documentation in the light-doc repository for the changes if necessary. For community contributors, it is optional, and we will help to update the document during the merging process. 

Perhaps you only want to start contributing to the documents. If so, you can ignore most of the above steps and focus on the `light-doc` repository. 

You can start Hugo's built-in server via `hugo server`. Browse the documentation by entering [http://localhost:1313](http://localhost:1313) in the address bar of your browser. The server automatically updates the page whenever you change content.

We have developed a [separate documentation contribution guide][docscontrib] for more information on how the Hugo docs are built, organized, and improved by the generosity of people like you.

### Where to start?

Thank you for reading through this contribution guide. Hopefully, we will see you again soon on GitHub. There are plenty of [open issues][issues] for you to help with.

Feel free to open an issue if you think you found a bug or you have a new idea to improve the light platform. We are happy to hear from you.

### Additional References for Learning Git

* [Codecademy's Free "Learn Git" Course][codecademy] (Free)
* [Code School and GitHub's "Try Git" Tutorial][trygit] (Free)
* [The Git Book][gitbook] (Free)
* [GitHub Pull Request Tutorial, Thinkful][thinkful]


[codecademy]: https://www.codecademy.com/learn/learn-git
[request for comments]: https://github.com/networknt/light-rfcs
[issues]: /contribute/issues/
[docscontrib]: /contribute/documentation/
[light-bot tutorial]: /tutorial/bot/local-develop/
[light-bot]: /tool/light-bot/
[gitbook]: https://git-scm.com/
[thinkful]: https://www.thinkful.com/learn/github-pull-request-tutorial/
[trygit]: https://try.github.io/levels/1/challenges/1
