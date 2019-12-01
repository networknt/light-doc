---
title: "RcSelect Developer Guide"
date: 2019-11-30T22:39:57-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Recently, I have upgraded this react-schema-form add-on to the latest for all dependencies to resolve the security vulnerabilities. The process is relatively easy. 

To update all packages to the latest major version, I followed this [document](https://flaviocopes.com/update-npm-dependencies/) on the parent folder. 

For the RcSelect.js, I have switched jquery to fetch for the data fetching to remove the dependency for the jquery. Also, the componentWillMount is renamed to componentDidMount according to the React warning message. 

Almost all supporting files have been updated or synced from the react-schema-form repository. 

### Example

The example folder was removed and regenerated with create-react-app and copied over the existing code. 

It will streamline future development and upgrade process as most people are very familiar with the create-react-app applications. 

To debug the unpublished RcSelect, the dependency for the react-schema-form-rc-select is defined as below in the package.json in the example folder. 

```
"react-schema-form-rc-select": "file:../",
```

Once the RcSelect.js is updated, you need to run the `npm run prepublish` first to build the final version in the lib folder. And then, you can use `npm install` or `yarn install` in the example folder after removing the node-modules folder. 

It is very time consuming, but I haven't found any shortcut to utilize the source code from the parent src folder directly in the example yet. 


### Demo Site

The demo site is deployed to the gh-pages branch in the same repository. To do that, you need to build the example with the following command. 

```
cd example
rm -rf node-modules
yarn install
```

Once all dependencies are installed, you can set up the gh-pages deploy command by following this [tutorial](https://github.com/gitname/react-gh-pages)

Once it is set up, the only command you need to do is to run the `npm run deploy` and the latest code will be checked into the gh-pages branch for the same repository from the example folder. 


