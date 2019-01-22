---
title: "Portal View Tutorial"
date: 2019-01-03T19:39:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This is a very long tutorial that documents the steps we build light-portal UI  with React and other related libraries. It is not part of the core features of our [light-4j platform][]; however, I think it is essential for the following reasons. 

Most of our developers are backend focused, and the front end Javascript is not the strengths for us. When starting the UI for the portal, I am going to document the steps so that our team member can learn from the tutorial. Also, for contributors, it is an essential document to learn how the UI is constructed before contributing. For the customers who are deploying the light-portal on-premise with customizations or with customized view on our virtual host cloud, this document will give them an entry point to start their work. 

I have used the earlier versions of React in 2015 and created [react-schema-form] to ease the pain for UI form development. Since then, most of my focus is on the light-4j platform in Java. The entire landscape of React ecosystem changed a lot, and I am in a learning process. By writing the tutorial, I am hoping some experts can point out my errors and provide recommendations for improvement. 

While re-learning React, I found there are a lot of tutorials focus on a specific area; however, it is tough to find one that glues everything together. For example, how to integrate React, Redux, Router with Async API interactions.  This tutorial is aiming to document all the small details so that developers can have a big picture of the interactions between components. 

As this is a live project, the scope is not fixed and we might add certain topics according to readers' feedbacks.  Basically, we want to build the portal UI as front-end microservices so that we can add more services once they are available without major refactor the UI. React is our choice of the library as it is component based and very flexible. Other libraries used in the view are react-redux, material-ui, react-router and [react-schema-form][] etc.


* [create-react-app][] - API-Certification Service
* [build and deploy][] - Build and deploy the portal view
* [Jest Test Runner][] - Hook up test runner to watch file changes
* [Wire in Redux][] - Wire in Redux Provider for state store
* [Add material-ui][] - Add masterial-ui for React components
* [Add react-router][] - Add react-router for routing in SPA
* [Tree View Menu][] - Make dynamic tree view menu with local definition file

[react-schema-form]: https://github.com/networknt/react-schema-form
[create-react-app]: /tutorial/portal/view/create-react-app/
[build and deploy]: /tutorial/portal/view/build-deploy/
[light-4j platform]: /about/what-is-light/
[Jest Test Runner]: /tutorial/portal/view/jest-test-runner/
[Wire in Redux]: /tutorial/portal/view/wire-in-redux/
[Add material-ui]: /tutorial/portal/view/add-material-ui/
[Add react-router]: /tutorial/portal/view/add-react-router/
[Tree View Menu]: /tutorial/portal/view/tree-view-menu/

