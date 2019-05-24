---
title: "Jest Test Runner"
date: 2019-01-04T09:19:31-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 30
aliases: []
toc: false
draft: false
---

In the previous tutorial [build and deploy][], we have scaffold the view application, built and deployed it to the light-portal site. Before we start adding code to the project, we are going to hook up the Jest Test Runner to watch the file changes and run tests automatically in the background. 

For a simple application, you can always test it on the browser manually after each change; however, for a large application, it is much more efficient to test with unit and integration test cases. 

Writing test cases takes some time, but you will save a lot of time in the long run. For best practice, we want to let the test runner in the background. 

Open a terminal to start test runner.

```
cd ~/networknt/light-portal/view
npm run test
```

Press `a` to run all test and watch the file system for change. Once you have updated source code, the test running will automatically run all test cases. It would be good to keep this terminal and the browser on one side of the screen and your code editor on the other side. Once you change the code, you can see the result on both the browser and the test terminal. 

The newly scaffolded application only has one test case and here is the output after press `a`. 

```
 PASS  src/App.test.js
  ✓ renders without crashing (2ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.109s, estimated 1s
Ran all test suites.

Watch Usage: Press w to show more.
```

Now the test runner is watching the file change and will rerun the test cases once that happens.

In the following tutorials, we will add more unit and integration test cases. In the next tutorial, we are going to [wire in Redux][] into our application to manage the state. 

[wire in Redux]: /tutorial/portal/view/wire-in-redux/
[build and deploy]: /tutorial/portal/view/build-deploy/
