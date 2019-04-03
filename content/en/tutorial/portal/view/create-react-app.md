---
title: "Create React App"
date: 2019-01-03T20:23:20-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

The first step is to scaffold the react application, and the easiest way is to do it with `create-react-app` command line tool provided by Facebook. 

First, you need to ensure that you have Nodejs installed on your computer. Also, you need to choose a workspace for your project. I am creating a view subfolder in the light-portal repository to group the UI and backend services for easy management. 

```
cd ~/networknt/light-portal
npx create-react-app view
```

After the project is created and all dependencies are installed, you can go to the view folder and start the application.

```
cd view
yarn start
```

Once the application starts, it will start a new tab on your browser with a logo on a simple page. Let's update the application a little bit and observe the change on the browser. 

Open the view folder from IntelliJ IDEA (You can use visual studio code if you don't have an IDEA license). 

On the UI, there is a text `Edit src/App.js and save to reload.` and we are going to change it to `Light API Portal`

First, locate the App.js in the src folder and open it. 

Replace the following line

```
Edit <code>src/App.js</code> and save to reload.
```

To 

```
Light API Portal
```

Save the file and go back to the browser, you should see the application is updated with the `Light API Portal` displayed on the screen. React Hot Loader provides this feature for productive for UI development. 

Now let's check in the view to branch `ui-create-react-app` for the light-portal repository. For further updates, you can compare with this branch to see what has been changed in each step. 

In the next step, we are going to [build and deploy][] the view app to the light-portal server as part of the virtual host. 


[build and deploy]: /tutorial/portal/view/build-deploy/
