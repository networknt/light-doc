---
title: "Add React Router"
date: 2019-01-04T15:21:01-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 60
aliases: []
toc: false
draft: false
---

The light-portal has a lot of services and that means there would be a lot of menus on the UI and we need react-route to manage the routing in between components. 

### Dependencies

Run the following command to install the react-router.

```
cd ~/networknt/light-portal/view
yarn add react-router react-router-dom
```

If you are using npm.

```
npm install --save react-router react-router-dom
```

After the installation, the dependencies in package.json should look like this. 

```
  "dependencies": {
    "@material-ui/core": "^3.8.1",
    "@material-ui/icons": "^3.0.1",
    "react": "^16.7.0",
    "react-dom": "^16.7.0",
    "react-redux": "^6.0.0",
    "react-router": "^4.3.1",
    "react-router-dom": "^4.3.1",
    "react-scripts": "2.1.2",
    "redux": "^4.0.1",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.3.0"
  },
```

The vesions might be different when you follow the tutorial yourself. 

### ResponsiveDrawer

In order to wire in the react-router, let's first create a component called ResponsiveDrawer to handler the hard-coded menu. 

Create a file ResponsiveDrawer.js in components. 

```

```



