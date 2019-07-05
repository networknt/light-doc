---
title: "Wire in Redux"
date: 2019-01-04T09:40:55-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 40
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous tutorial, we have hooked up the [Jest Test Runner][] and watch the file change to rerun all test cases. In this tutorial, we are going to wire in Redux to manage the state of the application. 

### Install Redux

First we need to add dependencies. 

```
cd ~/networknt/light-portal/view
yarn add redux react-redux redux-thunk redux-logger
```

If you are using npm, then you can run

```
npm install --save redux react-redux redux-thunk redux-logger
```

After it is done, you should see the following dependencies in your package.json

```
  "dependencies": {
    "react": "^16.7.0",
    "react-dom": "^16.7.0",
    "react-redux": "^6.0.0",
    "react-scripts": "2.1.2",
    "redux": "^4.0.1",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.3.0"
  },
```

As the entire React ecosystem is changing very fast, the versions of each component might be different for you. 

### Files Structure

With Redux in the picture, we need to re-organize our project structure to make it easier to add new components later.

Let's create the following folders in the src.

* components 
* reducers
* actions


### Home Component

Let's create the first component Home.js in components folder as our landing page. 

```
import React, {Component} from 'react';

class Home extends Component {
    render() {
        return (
            <div>
                Home
            </div>
        )
    }
}

export default Home;
```

### Reducers

We are planning to load menu for each host from our backend serivce so that each host can customized which applications can be accessed through menus. So when application is started, we should have a state that contains a list of menu items. Let's create a reducer named menu.js as following. 


```
export default function(state = [], action) {
    switch (action.type) {
        default:
            return state;
    }
}
```

As you can see, the function takes a state with an empty list as default and an action for parameters. As we don't have any action yet, the default in switch block just return the current state. 

Now, let's create an index.js in the reducers folder as Root Reducer that can combine all other reducers together. 


```
import { combineReducers } from 'redux';
import menuReducer from './menu';

export default combineReducers({
    menu: menuReducer
});
```

For now, we only have one reducer and more will be added later. 

### Provider

Now, let's update the index.js in the src folder to wire in the Proivider tag. Open the file from src folder and add the following imports. 

```
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import rootReducer from './reducers';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
```

Add the following const.

```
const store = createStore(rootReducer, applyMiddleware(thunk, logger));
```

And update the render to. 

```
ReactDOM.render(
    <Provider store={store}>
      <App />
    </Provider>,
    document.getElementById('root')
);
```

### Actions

In the actions folder, we are going to create a types.js and an index.js

types.js

```
export const LOAD_MENU = 'load_menu';
```

index.js

```
import { LOAD_MENU } from './types';

export function loadMenu(host) {
    return {
        type: LOAD_MENU,
        payload: host
    }
}
```

Now, double check the application still running without any error from the console tab of developers tool. 


At this stage, we have wired in Redux to our application. We still need to update the components later to connect them with Redux store for states and actions. 

Let's check in the code to branch `ui-wire-in-redux` for this tutorial and in the next tutorial, we are going to [add material-ui][] to our application. 

```
git checkout -b ui-wire-in-redux
git add .
git commit -m "ui view wire in redux"
git push origin ui-wire-in-redux
```


[Jest Test Runner]: /tutorial/portal/view/jest-test-runner/
[add material-ui]: /tutorial/portal/view/add-material-ui/
