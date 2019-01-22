---
title: "Tree View Menu"
date: 2019-01-21T16:32:30-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 70
aliases: []
toc: false
draft: false
---

In the previous section, we have [added react-router][] into the application with a responsive drawer implemented. We also added two menu links `Home` and `Abort` to route to these components. As we are building a portal site,  we might have a lot of services integrated into the portal with menus. Also, we are planning to support multi-tenant on the site so that each host can define its customized menu items to choose which services to be available. 

Given the above requirement, we need to have a service to manage menu items per host in a hierarchical structure and serve the menu items when the Single Page Application is loaded for the host. To make it a little bit simpler, we want to build the tree view menu first with a static JSON definition file locally. In the following sections, we are going to hook up with the menu service of the light-portal to get JSON response from the service by the host. 

The tree view is customized from https://github.com/fiffty/react-treeview-mui project with Rect 16 and Material UI 3.x support. As the project is only an old version released to the npmjs.org, we had to copy the component and made the modifications. 

Here is the MuiTreeList.js

```
import React from 'react';
import PropTypes from 'prop-types';
import {Link} from 'react-router-dom';
import { List, ListItem, ListItemText, ListItemIcon } from '@material-ui/core';
import OpenIcon from '@material-ui/icons/ExpandMore';
import CloseIcon from '@material-ui/icons/ExpandLess';
import FolderIcon from '@material-ui/icons/Folder';
import FileIcon from '@material-ui/icons/InsertDriveFile';

class TreeList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      expandedListItems: [],
      activeListItem: null,
    };
    this.handleTouchTap = this.handleTouchTap.bind(this);
  }

  handleTouchTap(listItem, index) {
    if (listItem.children) {
      const indexOfListItemInArray = this.state.expandedListItems.indexOf(index);
      if (indexOfListItemInArray === -1) {
        this.setState({
          expandedListItems: this.state.expandedListItems.concat([index]),
        });
      } else {
        let newArray = [].concat(this.state.expandedListItems);
        newArray.splice(indexOfListItemInArray, 1);
        this.setState({
          expandedListItems: newArray,
        });
      }
    } else {
      this.setState({
        activeListItem: index,
      });
    }
  }

  render() {
    // required props
    const { children, listItems, contentKey } = this.props;
    // optional props
    const style = this.props.style ? this.props.style : {};
    const startingDepth = this.props.startingDepth ? this.props.startingDepth : 1;
    const expandedListItems = this.props.expandedListItems
      ? this.props.expandedListItems
      : this.state.expandedListItems;
    const activeListItem = this.props.activeListItem
      ? this.props.activeListItem
      : this.state.activeListItem;
    const listHeight = this.props.listHeight ? this.props.listHeight : '48px';

    let listItemsModified = listItems.map((listItem, i, inputArray) => {
      listItem._styles = {
        root: {
          paddingLeft: (listItem.depth - startingDepth) * 16,
          backgroundColor: activeListItem === i ? 'rgba(0,0,0,0.2)' : null,
          height: listHeight,
          cursor: listItem.disabled ? 'not-allowed' : 'pointer',
          color: listItem.disabled ? 'rgba(0,0,0,0.4)' : 'rgba(0,0,0,0.87)',
          overflow: 'hidden',
          transform: 'translateZ(0)',
        },
      };
      return listItem;
    });
    listItemsModified = listItemsModified.map((listItem, i) => {
      listItem._shouldRender = listItem.depth >= startingDepth && parentsAreExpanded(listItem);
      listItem._primaryText = listItem[contentKey];
      return listItem;
    });

    // JSX: array of listItems
    const listItemsJSX = listItemsModified.map((listItem, i) => {
      if (listItem._shouldRender) {
        var inputProps = {
          key : 'treeListItem-' + i,
          style: listItem._styles.root
        };
        if(listItem.link) {
          inputProps.component=Link;
          inputProps.to = listItem.link;
        }
        return (
          <ListItem
            {...inputProps}
            onClick={() => {
              if (listItem.disabled) return;
              this.handleTouchTap(listItem, i);
            }}
            button
          >
            <ListItemIcon>{getLeftIcon(listItem, this.props.useFolderIcons)}</ListItemIcon>
            <ListItemText primary={listItem._primaryText} />
            {!listItem.children ? null : expandedListItems.indexOf(i) === -1 ? (
              <OpenIcon />
            ) : (
              <CloseIcon />
            )}
          </ListItem>
        );
      } else {
        return null;
      }
    });

    // styles for entire wrapper
    const styles = {
      root: {
        padding: 0,
        paddingBottom: 8,
        paddingTop: children ? 0 : 8,
      },
    };
    return (
      <div style={Object.assign({}, styles.root, style)}>
        {children}
        <List>
          {listItemsJSX}
        </List>
      </div>
    );

    function getLeftIcon(listItem, useFolderIcons) {
      if (useFolderIcons) {
        if (listItem.children) {
          return <FolderIcon />;
        } else {
          return <FileIcon />;
        }
      } else {
        return listItem.icon;
      }
    }

    function parentsAreExpanded(listitem) {
      if (listitem.depth > startingDepth) {
        if (expandedListItems.indexOf(listitem.parentIndex) === -1) {
          return false;
        } else {
          const parent = listItems.filter((_listItem, index) => {
            return index === listitem.parentIndex;
          })[0];
          return parentsAreExpanded(parent);
        }
      } else {
        return true;
      }
    }
  }
}

TreeList.contextTypes = {
  muiTheme: PropTypes.object,
};

TreeList.propTypes = {
  activeListItem: PropTypes.number,
  children: PropTypes.any,
  contentKey: PropTypes.string.isRequired,
  expandedListItems: PropTypes.array,
  handleTouchTap: PropTypes.func,
  listHeight: PropTypes.number,
  listItems: PropTypes.array.isRequired,
  startingDepth: PropTypes.number,
  style: PropTypes.object,
  useFolderIcons: PropTypes.bool,
};

export default TreeList;

```

This component renders a list of menu items in a tree-like structure. All the menu items are in a list and rendered with different depth and visibility. Only the leaf nodes have links for routing, so we need to do a trick to render all items as some list items won't have `component` and `to` props. We first define a const for the props and conditional set component and to based on if the link exists or not.

```
        var inputProps = {
          key : 'treeListItem-' + i,
          style: listItem._styles.root
        };
        if(listItem.link) {
          inputProps.component=Link;
          inputProps.to = listItem.link;
        }
```

Next we use a spread to put the props into the component. 

```
          <ListItem
            {...inputProps}
            onClick={() => {
              if (listItem.disabled) return;
              this.handleTouchTap(listItem, i);
            }}
            button
          >
```

The TreeList uses a ListItem component which is in ListItem.js

```
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class ListItem extends Component {
  render() {
    const { primaryText, style } = this.props;
    const { onTouchTap, leftIcon } = this.props;

    const styles = {
      root: {
        cursor: 'pointer',
        transition: 'all 0.25s ease-in-out',
      },
      primaryText: {
        lineHeight: '32px',
      },
    };

    return (
      <div style={Object.assign({}, styles.root, style)} onClick={onTouchTap}>
        {leftIcon}
        <span style={Object.assign({}, styles.primaryText)}>{primaryText}</span>
      </div>
    );
  }
}

ListItem.propTypes = {
  primaryText: PropTypes.string.isRequired,
  style: PropTypes.object.isRequired,
  leftIcon: PropTypes.element,
  rightIcon: PropTypes.element,
  to: PropTypes.string,
  onTouchTap: PropTypes.func,
};

export default ListItem;

```

With these components done, we need to update the existing ResponsiveDrawer to use the MuiTreeList instead of static menus. 

With three levels of the tree, we need to update the width of the drawer to 300 from 240. 

```
const drawerWidth = 300;
```
We need to pass the listItems to the MuiTreeLIst as props and here is the const definitions. 

```
const {listItems} = this.state;
const { classes, children, theme } = this.props;
const treeList = (
     <MuiTreeList
          listItems={listItems}
          contentKey={'title'}
          useFolderIcons={true}
     >
     </MuiTreeList>
 );
```

Here is the entire ResponsiveDrawer.js

```
import React, {Component} from 'react';
import {withRouter} from 'react-router-dom';
import {compose} from 'recompose';
import PropTypes from 'prop-types';
import { withStyles } from '@material-ui/core/styles';
import Drawer from '@material-ui/core/Drawer';
import AppBar from '@material-ui/core/AppBar';
import Toolbar from '@material-ui/core/Toolbar';
import CssBaseline from '@material-ui/core/CssBaseline';
import Typography from '@material-ui/core/Typography';
import IconButton from '@material-ui/core/IconButton';
import Hidden from '@material-ui/core/Hidden';
import MenuIcon from '@material-ui/icons/Menu';
import listItems from '../data/ListItems';
import MuiTreeList from './MuiTreeList';
const drawerWidth = 300;

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
        expandedListItems: [],
        listItems: listItems,
    };

    handleDrawerToggle = () => {
        this.setState(state => ({ mobileOpen: !state.mobileOpen }));
    };

    render() {
        const {listItems} = this.state;
        const { classes, children, theme } = this.props;
        const treeList = (
            <MuiTreeList
                listItems={listItems}
                contentKey={'title'}
                useFolderIcons={true}
            >
            </MuiTreeList>
        );
        const drawer = (
            <div>
                <div className={classes.toolbar} />
                {treeList}
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

To make the menu component to work, we need to create a JSON formatted data file under data directory from the src. Here is the ListItems.json

```
[
  {
    "depth":0,
    "children":[

    ],
    "_styles":{
      "root":{
        "paddingLeft":-16,
        "backgroundColor":null,
        "height":"48px",
        "cursor":"pointer",
        "color":"rgba(0,0,0,0.87)",
        "overflow":"hidden",
        "transform":"translateZ(0)"
      }
    },
    "_shouldRender":false
  },
  {
    "title":"OAuth",
    "depth":1,
    "parentIndex":0,
    "children":[
      2,
      5
    ],
    "disabled":false
  },
  {
    "title":"PlayGround",
    "depth":2,
    "children":[
      3,
      4
    ],
    "parentIndex":1,
    "disabled":false
  },
  {
    "title":"Long-lived Token",
    "depth":3,
    "parentIndex":2,
    "disabled":false,
    "link":"/test/long-token"
  },
  {
    "title":"Client Registration",
    "depth":3,
    "parentIndex":2,
    "disabled":false,
    "link":"/test/client/register"
  },
  {
    "title":"Production",
    "depth":2,
    "children":[
      6,
      7
    ],
    "disabled":false,
    "parentIndex":1
  },
  {
    "title":"Client Registration",
    "depth":3,
    "parentIndex":5,
    "disabled":false,
    "link":"/prod/client/register"
  },
  {
    "title":"Service Registration",
    "depth":3,
    "parentIndex":5,
    "disabled":false,
    "link":"/prod/service/register"
  },
  {
    "title":"MarketPlace",
    "depth":1,
    "parentIndex":0,
    "disabled":false,
    "children":[
      9,
      12
    ]
  },
  {
    "title":"REST",
    "depth":2,
    "parentIndex":8,
    "disabled":false,
    "children":[
      10,
      11
    ]
  },
  {
    "title":"API List",
    "depth":3,
    "parentIndex":9,
    "disabled":false,
    "link":"/rest/list"
  },
  {
    "title":"Register",
    "depth":3,
    "parentIndex":9,
    "disabled":false,
    "link":"/rest/register"
  },
  {
    "title":"Hybrid",
    "depth":2,
    "parentIndex":8,
    "disabled":false,
    "children":[
      13,
      14
    ]
  },
  {
    "title":"API List",
    "parentIndex":12,
    "depth":3,
    "disabled":true,
    "link":"/hybrid/list"
  },
  {
    "title":"Register",
    "parentIndex":12,
    "depth":3,
    "disabled":true,
    "link":"/hybrid/register"
  },
  {
    "title":"About",
    "depth":1,
    "parentIndex":0,
    "disabled":false,
    "link":"/about"
  }
]

```

Now, if we go to the browser, we can expand the menus, and if you click any leaf menu items, the routing path is changed on the browser address bar. Since we don't have all the links implemented, only `About` menu will route the page to the `About` page at the moment. 

Let's check in the source code into the ui-tree-view-menu branch so that users can compare with the previous branches to see the differences. 

```
git checkout -b ui-tree-view-menu
git add .
git commit -m "tree view menu with local file"
git push origin ui-tree-view-menu

```

In the next section, we are going to add [react-schema-form][] component to have a generic form generation and handling for the majority of the pages. 


[added react-router]: /tutorial/portal/add-react-router/
[react-schema-form]: /tutorial/portal/react-schema-form/

