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
reviewed: true
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
import React, {Component} from 'react';
import {Link, withRouter} from 'react-router-dom';
import {compose} from 'recompose';
import PropTypes from 'prop-types';
import { withStyles } from '@material-ui/core/styles';
import Drawer from '@material-ui/core/Drawer';
import AppBar from '@material-ui/core/AppBar';
import Toolbar from '@material-ui/core/Toolbar';
import CssBaseline from '@material-ui/core/CssBaseline';
import MenuList from '@material-ui/core/MenuList';
import Typography from '@material-ui/core/Typography';
import IconButton from '@material-ui/core/IconButton';
import Hidden from '@material-ui/core/Hidden';
import Divider from '@material-ui/core/Divider';
import MenuIcon from '@material-ui/icons/Menu';
import MenuItem from '@material-ui/core/MenuItem';

const drawerWidth = 240;

const styles = theme => ({
    root: {
        display: 'flex',
    },
    drawer: {
        [theme.breakpoints.up('sm')]: {
            width: drawerWidth,
            flexShrink: 0,
        },
    },
    appBar: {
        marginLeft: drawerWidth,
        [theme.breakpoints.up('sm')]: {
            width: `calc(100% - ${drawerWidth}px)`,
        },
    },
    menuButton: {
        marginRight: 20,
        [theme.breakpoints.up('sm')]: {
            display: 'none',
        },
    },
    toolbar: theme.mixins.toolbar,
    drawerPaper: {
        width: drawerWidth,
    },
    content: {
        flexGrow: 1,
        padding: theme.spacing.unit * 3,
    },
});


class ResponsiveDrawer extends Component {
    state = {
        mobileOpen: false,
    };

    handleDrawerToggle = () => {
        this.setState(state => ({ mobileOpen: !state.mobileOpen }));
    };

    render() {
        const { classes, children, location: { pathname }, theme } = this.props;
        const drawer = (
            <div>
                <div className={classes.toolbar} />
                <Divider />
                <MenuList>
                    <MenuItem component={Link} to="/" selected={'/' === pathname}>
                        Home
                    </MenuItem>
                    <MenuItem component={Link} to="/about" selected={'/about' === pathname}>
                        About
                    </MenuItem>
                </MenuList>
            </div>
        );

        return (
            <div className={classes.root}>
                <CssBaseline />
                <AppBar position="fixed" className={classes.appBar}>
                    <Toolbar>
                        <IconButton
                            color="inherit"
                            aria-label="Open drawer"
                            onClick={this.handleDrawerToggle}
                            className={classes.menuButton}
                        >
                            <MenuIcon />
                        </IconButton>
                        <Typography variant="h6" color="inherit" noWrap>
                            API Portal
                        </Typography>
                    </Toolbar>
                </AppBar>
                <nav className={classes.drawer}>
                    {/* The implementation can be swap with js to avoid SEO duplication of links. */}
                    <Hidden smUp implementation="css">
                        <Drawer
                            container={this.props.container}
                            variant="temporary"
                            anchor={theme.direction === 'rtl' ? 'right' : 'left'}
                            open={this.state.mobileOpen}
                            onClose={this.handleDrawerToggle}
                            classes={{
                                paper: classes.drawerPaper,
                            }}
                            ModalProps={{
                                keepMounted: true, // Better open performance on mobile.
                            }}
                        >
                            {drawer}
                        </Drawer>
                    </Hidden>
                    <Hidden xsDown implementation="css">
                        <Drawer
                            classes={{
                                paper: classes.drawerPaper,
                            }}
                            variant="permanent"
                            open
                        >
                            {drawer}
                        </Drawer>
                    </Hidden>
                </nav>
                <main className={classes.content}>
                    <div className={classes.toolbar} />
                    {children}
                </main>
            </div>
        );
    }
}


ResponsiveDrawer.propTypes = {
    classes: PropTypes.object.isRequired,
    // Injected by the documentation to work in an iframe.
    // You won't need it on your project.
    container: PropTypes.object,
    theme: PropTypes.object.isRequired,
};

export default compose (
    withRouter,
    withStyles(styles, { withTheme: true })
)(ResponsiveDrawer);

```

In the MenuList, we have two MenuItems: Home -> / and About -> /about as hard-coded menu for now. 

We have Home.js in the components folder already, let's add About.js for the /about page. It is almost the same as Home but changes the text to `About` from `Home`.

About.js


```
import React, {Component} from 'react';

class About extends Component {
    render() {
        return (
            <div>
                About
            </div>
        )
    }
}

export default About;

```

Now let's wire the react-router in App.js

```
import React, { Component } from 'react';
import {Switch, Route} from 'react-router-dom';
import { Router } from 'react-router';
import createBrowserHistory from 'history/createBrowserHistory'
import Home from './components/Home';
import About from './components/About';
import ResponsiveDrawer from './components/ResponsiveDrawer';

export const history = createBrowserHistory();

class App extends Component {
  render() {
    return (
        <Router history={history}>
          <ResponsiveDrawer>
            <Switch>
              <Route exact path="/" component={Home} />
              <Route path="/about" component={About} />
            </Switch>
          </ResponsiveDrawer>
        </Router>
    );
  }
}

export default App;

```

Note that the newly added imports and the history export. We also change the render to use Router to wrap the ResponsiveDrawer. 


Now if you go to the browser, you can see the menu shows up on the left of the screen. You can click `Home` or `About` to see that the page is switching between Home and About. 

When you narrow the browser window, you can see that the drawer is going with a menu button on the upper left corner. You click the button, the drawer will show up with the menu, and once you choose one of the menus, the view will be switched, and the drawer will disappear again. 

Before moving to the next tutorial, let's check in the changes to a branch called `ui-add-react-router`

```
git checkout -b ui-add-react-router
git add .
git commit -m "add react-router"
git push origin ui-add-react-router
```


At this stage, we have react-router works and a static menu displays on a responsive drawer. In the next step, we are going to change the flattened menu to [tree view menu][] with a static json definition. 

[tree view menu]: /tutorial/portal/view/tree-view-menu/
