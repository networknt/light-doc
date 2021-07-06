---
title: "Add material-ui"
date: 2019-01-04T13:24:42-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 50
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous tutorial, we have [wired in Redux][] into the application. Bofore adding any component, we want to add [material-ui][] into the picture. 

### Dependencies

Let's go to the view folder and add the dependencies. 

```
cd ~/networknt/light-portal/view
yarn add @material-ui/core @material-ui/icons
```

If you are using npm. 

```
npm install --save @material-ui/core @material-ui/icons
```

Once the material-ui is installed, your dependencies should look like the following in package.json

```
  "dependencies": {
    "@material-ui/core": "^3.8.1",
    "@material-ui/icons": "^3.0.1",
    "react": "^16.7.0",
    "react-dom": "^16.7.0",
    "react-redux": "^6.0.0",
    "react-scripts": "2.1.2",
    "redux": "^4.0.1",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.3.0"
  },
```

The versions might be different if you follow the tutorial at a later date. 


### MuiThemeProvider

Now, we are going to update the src/index.js to add MuiThemeProvider. 

Open the file and add the following imports. 

```
import { MuiThemeProvider, createMuiTheme } from '@material-ui/core/styles';
```

Create a const theme.

```
const theme = createMuiTheme({
    typography: {
        useNextVariants: true,
    },
});
```

This will remove a lot of warning messages in the console. 


Wrap the <App /> with the provider. 

```
ReactDOM.render(
    <Provider store={store}>
        <MuiThemeProvider theme={theme}>
            <App />
        </MuiThemeProvider>
    </Provider>,
    document.getElementById('root')
);
```

As you can see that we have wrapped the <App /> with two level of providers now. 

Before moving to the next tutorial, let's check in the changes to a branch called `ui-add-material-ui`

```
git checkout -b ui-add-material-ui
git add .
git commit -m "add material-ui"
git push origin ui-add-material-ui
```

In the next tutorial, we are going to [add react-router][] to the App component to handler the routing in the single page application. 


[wired in Redux]: /tutorial/portal/view/wire-in-redux/
[material-ui]: https://material-ui.com/
[add react-router]: /tutorial/portal/view/add-react-router/

