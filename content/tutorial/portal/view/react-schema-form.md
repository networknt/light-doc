---
title: "React Schema Form"
date: 2019-01-20T18:19:36-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the previous [tree-view-menu][] section, we have added several menu items into the ResponsiveDrawer to form a hierarchical navigation component with react-router integrated. In this section, we are going to hook up [react-schema-form][] to have generic form generation for most of the menu items. 

The react-schema-form is a react component we built on top of material-ui to help us to build SPA that interacts with backend services built on top of [light-hybrid-4j][] and [light-rest-4j][]. To learn more about react-schema-form, you can find an online [demo](http://networknt.github.io/react-schema-form). If you want to try a real application, please take a look at the [Taiji Blockchain Web Client](https://demo.taiji.io/).

### Dependencies

Run the following command to install the react-schema-form.

```
cd ~/networknt/light-portal/view
yarn add react-schema-form
```

If you are using npm.

```
npm install --save react-schema-form
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
    "react-schema-form": "^0.6.10",
    "react-scripts": "2.1.2",
    "redux": "^4.0.1",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.3.0"
  },
 ```

The vesions might be different when you follow the tutorial yourself. 

### Form Component

Now let's create a new form component as Form.js

```
import React, {Component} from 'react';
import { withStyles } from '@material-ui/core/styles';
import { submitForm} from "../actions";
import connect from "react-redux/es/connect/connect";
import CircularProgress from '@material-ui/core/CircularProgress';
import Button from '@material-ui/core/Button';
import SchemaForm from 'react-schema-form/lib/SchemaForm';
import utils from 'react-schema-form/lib/utils';
import forms from '../data/Forms';

const styles = theme => ({
    root: {
        display: 'flex',
        flexWrap: 'wrap',
    },
    formControl: {
        margin: theme.spacing.unit,
        minWidth: 120,
    },
    selectEmpty: {
        marginTop: theme.spacing.unit * 2,
    },
    progress: {
        margin: theme.spacing.unit * 2,
    },
    button: {
        margin: theme.spacing.unit,
    },
});

class Form extends Component {

    constructor(props) {
        super(props);
        this.state = {
            fetching: false,
            error: null,
            formId: null,
            schema: null,
            form: null,
            actions: null,
            model: {}
        }
    }

    componentWillUpdate(nextProps, nextState, nextContext) {
        if(this.state.formId !== nextProps.match.params.formId) {
            let formData = forms[this.props.match.params.formId];
            this.setState({
                formId: nextProps.match.params.formId,
                schema: formData.schema,
                form: formData.form,
                actions: formData.actions,
                model: formData.model || {}
            });
        }
    }

    componentDidMount() {
        let formData = forms[this.props.match.params.formId];
        this.setState({
            formId: this.props.match.params.formId,
            schema: formData.schema,
            form: formData.form,
            actions: formData.actions,
            model: formData.model || {}
        });
    }

    onModelChange = (key, val) => {
        //console.log(this.state.model);
        utils.selectOrSet(key, this.state.model, val);
    };

    onButtonClick(action) {
        console.log(action, this.state.model);
        let validationResult = utils.validateBySchema(this.state.schema, this.state.model);
        if(!validationResult.valid) {
            this.setState({error: validationResult.error.message});
        } else {
            action.data = this.state.model;
            this.setState({success: action.success, fetching: true});
            this.props.submitForm(action);
        }
    }

    render() {
        const { classes } = this.props;
        //console.log(this.state.actions);
        if(this.state.schema) {
            var actions = [];
            this.state.actions.map((item, index) => {
                let boundButtonClick = this.onButtonClick.bind(this, item);
                actions.push(<Button variant="contained" className={classes.button} color="primary" key={index} onClick={boundButtonClick}>{item.title}</Button>)
            });
            let wait;
            if(this.state.fetching) {
                //console.log('fetching is true');
                wait = <div><CircularProgress className={classes.progress} /></div>;
            } else {
                //console.log("fetching is false");
                wait = <div></div>;
            }
            let title = <h2>{this.state.schema.title}</h2>

            return (
                <div>
                    {wait}
                    {title}
                    <SchemaForm schema={this.state.schema} form={this.state.form} model={this.state.model} onModelChange={this.onModelChange} />
                    <pre>{this.state.error}</pre>
                    {actions}
                </div>
            )
        } else {
            return (<CircularProgress className={classes.progress} />);
        }
    }
}

const mapStateToProps = state => ({
});

const mapDispatchToProps = dispatch => ({
    submitForm: action => dispatch(submitForm(action))
});

export default connect(
    mapStateToProps,
    mapDispatchToProps
)(withStyles(styles)(Form));

```

### Form Data

This Form component maintains its states and loads a specific form by formId passed from the router path. The form is loaded from the local static file in data folder called Forms.json. Here is the content in the Forms.json

```
{
  "longLivedTokenForm": {
    "formId": "longLivedTokenForm",
    "actions": [
      {
        "host": "lightapi.net",
        "service": "playground",
        "action": "generateJwt",
        "version": "0.1.0",
        "title": "Generate Long-Lived Token",
        "success": "/playground/longLivedToken"
      }
    ],
    "schema": {
      "type": "object",
      "required": [
        "client_id",
        "scope"
      ],
      "title": "JWT Claims",
      "properties": {
        "exp": {
          "type": "integer"
        },
        "client_id": {
          "type": "string"
        },
        "user_id": {
          "type": "string"
        },
        "scope": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      }
    },
    "form": [
      "*"
    ]
  }
}
```

As you can see that it is used the same JSON schema published from the backend service to generate the form and it has a section to define actions for the submit buttons and the mapping service so that once a button is clicked, the corresponding service will be automatically invoked. 

All the JSON schemas for the light-portal services can be found at https://github.com/networknt/model-config/tree/master/hybrid

In the future, we are going to load the form from a service that is deployed on the portal query side service. Now, it is OK to put all forms into the same json file for development. 

### App

In the App.js we need to add one more route for path `/form/:formId` that is routed to the Form component. 

```
<Route path="/form/:formId" component={Form} />
```

Here is the updated App.js 

```
import React, { Component } from 'react';
import {Switch, Route} from 'react-router-dom';
import { Router } from 'react-router';
import createBrowserHistory from 'history/createBrowserHistory'
import Home from './components/Home';
import About from './components/About';
import ResponsiveDrawer from './components/ResponsiveDrawer';
import Form from './components/Form';

export const history = createBrowserHistory();

class App extends Component {
  render() {
    return (
        <Router history={history}>
          <ResponsiveDrawer>
            <Switch>
              <Route exact path="/" component={Home} />
              <Route path="/about" component={About} />
              <Route path="/form/:formId" component={Form} />
            </Switch>
          </ResponsiveDrawer>
        </Router>
    );
  }
}

export default App;

```

### Reducer

Now we need to create a FormReducer.js to handler the action types for form submission. As we are calling the backend service async, we need to handle three actions: STARTED, SUCCESS and FAILURE. 

Here is the FormReducer.js

```
import {SUBMIT_FORM_STARTED, SUBMIT_FORM_FAILURE, SUBMIT_FORM_SUCCESS} from "../actions/types";

const initialState = {};
export default (state = initialState, action) => {
    switch (action.type) {
        case SUBMIT_FORM_SUCCESS:
            return {
                ...state,
                fetching: false,
                result: action.payload
            };
        case SUBMIT_FORM_FAILURE:
            return {
                ...state,
                fetching: false,
                error: action.payload
            };
        case SUBMIT_FORM_STARTED:
            return {
                ...state,
                fetching: true
            };
        default:
            return state;
    }
}
```

With the new reducer added, we need to update the index.js in the reducers folder to add this new reducer. 

```
import { combineReducers } from 'redux';
import menuReducer from './menu';
import formReducer from './FormReducer';

export default combineReducers({
    menu: menuReducer,
    form: formReducer
});
```

### Action

Let's handle the actions for the form submission. First, we need to add the three actions to the types.js

```
export const LOAD_MENU = 'load_menu';
export const SUBMIT_FORM_STARTED = 'SUBMIT_FORM_STARTED';
export const SUBMIT_FORM_SUCCESS = 'SUBMIT_FORM_SUCCESS';
export const SUBMIT_FORM_FAILURE = 'SUBMIT_FORM_FAILURE';
```

Next, we need to add a function to handle the submitForm in the index.js in actions folder. 

```
export function submitForm(action) {
    return async (dispatch) => {
        dispatch({type: SUBMIT_FORM_STARTED});
        const request = {
            method: 'POST',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(action)
        };
        console.log(request);
        try {
            const response = await fetch('/api/portal', request);
            const data = await response.json();
            console.log("data", data);
            dispatch({ type: SUBMIT_FORM_SUCCESS, payload: data });
            history.push(action.success);
        } catch(e) {
            console.log("error " + e.toString());
            dispatch({ type: SUBMIT_FORM_FAILURE, error: e.toString()})
        }
    }
}

```

You can see that it async dispatch the call and await the fetch function to return. The backend service is using a relative url that means the single page application is served by the light-rotuer instance of light-protal. To learn more about how to use light-router for SPA, please visit [light-router tutorial][].

With above changes, you can expand the OAuth/PlayGround and click the `Long-lived Token` menu item. A `Jwt Claims` form will show up immediately. Once you fill in the form and click the generated `GENERATE LONG-LIVED TOKEN` button, the form will be submitted to the backend service. 


Now let's check in the code into a branch ui-react-schema-form. 

```
git checkout -b ui-react-schema-form
git add .
git commit -m "add react-schema-form"
git push origin ui-react-schema-form

```

In the next tutorial, we are going to setup a proxy in package.json and [wire in backend service][] to generate a real long lived token.

[tree-view-menu]: /tutorial/portal/view/tree-view-menu/
[react-schema-form]: https://github.com/networknt/react-schema-form
[light-hybrid-4j]: /style/light-hybrid-4j/
[light-rest-4j]: /style/light-rest-4j/
[light-router tutorial]: https://doc.networknt.com/tutorial/router/
[wire in backend service]: /tutorial/portal/view/wire-in-backend-service/

