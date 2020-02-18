---
title: "React App with long-lived token"
date: 2020-02-14T21:16:37-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous steps, we are using long-lived tokens to access services. With a single page application, we can also close the entire loop of the OAuth 2.0 authorization code flow. 

To start a React application, we need to create a repo on GitHub under the open-banking organization. Then clone the repo to the open-banking folder under the home directory. 

```
git clone git@github.com:open-banking/react-client.git
npx create-react-app react-client
yarn start
```

A browser tab will be started automatically after the node server is up with the SPA. Let's shut down the server and check in the generated code. 

```
cd ~/open-banking/react-client
git add .
git commit -m "initial checkin"
git push origin master
```

To test the client locally, we are going to set up the consul, light-oauth2, and light-router locally following this [local portal][] tutorial until the step to start the light-router. We also need to ensure that all open-banking services are up and running based on the previous steps. 

To make the ob.lightapi.net subdomain works locally, we need to update the /etc/hosts file to add this subdomain name. After the modification, we have the following line in the /etc/hosts file. 

```
192.168.1.144   obsignin.lightapi.net ob.lightapi.net
```

If you try the tutorial on your local, please change the IP address to your local address. You can find your local IP address with ifconfig on Linux.

With the generated react-client application, we need to make the following changes to make it work with a long-lived token. 

### useApiGet.js

This is a hook developed to access backend APIs from the single page application. 

```
import { useReducer, useEffect } from 'react';
import { requestStarted, requestSuccess, requestFailure } from './action';
import { reducer } from './reducer';
import { useCookies } from 'react-cookie';

export const useApiGet = ({ url, headers }) => {
  const [state, dispatch] = useReducer(reducer, {
    isLoading: true,
    data: null,
    error: null,
  });

  const [cookies, setCookie] = useCookies(['csrf']);
  console.log(cookies);
  Object.assign(headers, {'X-CSRF-TOKEN': cookies.csrf})
  console.log(headers);

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      dispatch(requestStarted());

      try {
        const response = await fetch(url, { headers, credentials: 'include', signal: abortController.signal });
        console.log(response);
        if (!response.ok) {
          throw new Error(
            `${response.status} ${response.statusText}`
          );
        }

        const data = await response.json();
        console.log(data);
        dispatch(requestSuccess({ data }));  
      } catch (e) {
        // only call dispatch when we know the fetch was not aborted
        if (!abortController.signal.aborted) {
          console.log(e.message);
          dispatch(requestFailure({ error: e.message }));
        }        
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, []);

  return state;
};

```

It accepts a URL and headers object as parameters. The useCookies hook is used to get the csrf cookie and put it into the headers. Please pay attention that credentials: 'include' is passed to the fetch API to ensure that the cookies will be sent along with the request to the server. 

### Accounts.js

We have 11 Apis, so we are going to pick up one to see how the component is used to invoke the useApiGet. All others are almost the same.

```
import React from 'react';
import { useApiGet } from '../hooks/useApiGet';
import CircularProgress from '@material-ui/core/CircularProgress';

export default function Accounts() {
  const url = 'https://ob.lightapi.net/accounts';
  const headers = {
    //Authorization: 'Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ',
    'x-fapi-financial-id': 'OB'
  };

  const { isLoading, data, error } = useApiGet({url, headers});

  let wait;
  if(isLoading) {
    wait = <div><CircularProgress/></div>;
  } else {
    wait = <div><pre>{ JSON.stringify(data, null, 2) }</pre></div>;
  }


  return (
    <div className="App">
      {wait}
    </div>
  );
}

```

Uncomment the Authorization in the headers to use the long-lived token even if the backend light-oauth2 services are not started. To make it simple, we display the result in a JSON format once it is retrieved. 

### ResponsiveDrawer.js

This component has the left menu list and a navbar with a login button. Once the button is clicked, the browser will be redirected to the obsignin.lightapi.net. 

```
    login() {
        window.location = "https://obsignin.lightapi.net?client_id=f7d42348-c647-4efb-a52d-4c5787421e74&user_type=customer&redirect_uri=https://ob.lightapi.net/authorization&state=1222";
    }

```


### Form

For most API calls, there are two endpoints. One list all the entities for the userId in the access token, and another will get the entity for a specific accountId. The accountId will be in the form of the path parameter. For example, /accounts/{AccountId}

To capture the accountId parameter, we are using the [react-schema-form][] to define the form in the Forms.json file. Here is the form for the accounts API. 

```
  "AccountsForm": {
    "formId": "AccountsForm",
    "actions": [
      {
        "host": "ob.lightapi.net",
        "title": "Filter",
        "success": "/accounts/"
      }
    ],
    "schema": {
      "type": "object",
      "required": [
        "accountId"
      ],
      "title": "Account Filter",
      "properties": {
        "userId": {
          "title": "User Id",
          "type": "string",
          "readonly": true
        },
        "accountId": {
          "title": "Account Id",
          "type": "string"
        }
      }
    },
    "form": [
      {
        "key": "accountId",
        "condition": "model.userId === 'stevehu'",
        "type": "select",
        "titleMap": [
          {
            "value": "22289",
            "name": "22289"
          },
          {
            "value": "31820",
            "name": "31820"
          }
        ]
      },
      {
        "key": "accountId",
        "condition": "model.userId === 'ericbroda'",
        "type": "select",
        "titleMap": [
          {
            "value": "42281",
            "name": "42281"
          },
          {
            "value": "41221",
            "name": "41221"
          }
        ]
      }
    ]
  },

```

There are a JSON schema and a form definition. In the form definition, we have conditionally defined the dropdown list based on the userId from the access token. 

### Form.js

This component loads the above Forms.json and displays the form to capture the user input. Here is the code to get the userId and put it into the model. 

```
    const [cookies, setCookie] = useCookies(['userId']);
    const userId = cookies.userId;
    console.log(userId);

    useEffect(() => {
        console.log(props.match.params.formId);
        let formData = forms[props.match.params.formId];
        console.log(formData);
        setSchema(formData.schema);
        setForm(formData.form);
        setActions(formData.actions);
        setModel(formData.model || {userId: userId})
    }, [props.match.params.formId])

```

Once the form button is clicked, the react-router is pushed to the /items as of following. 


```
            props.history.push({pathname: '/items', state: {path: action.success, accountId: model.accountId, statementId: model.statementId}});

```

### App.js

In the App.js, we define the Route. 

```
const App = () => {
    return (
          <ResponsiveDrawer>
            <Switch>
              <Route exact path="/" component={Home} />
              <Route exact path="/about" component={About} />
              <Route exact path="/accounts" component={Accounts} />
              <Route exact path="/balances" component={Balances} />
              <Route exact path="/transactions" component={Transactions} />
              <Route exact path="/beneficiaries" component={Beneficiaries} />
              <Route exact path="/directdebits" component={DirectDebits} />
              <Route exact path="/standingorders" component={StandingOrders} />
              <Route exact path="/products" component={Products} />
              <Route exact path="/offers" component={Offers} />
              <Route exact path="/parties" component={Parties} />
              <Route exact path="/scheduledpayments" component={ScheduledPayments} />
              <Route exact path="/statements" component={Statements} />
              <Route exact path="/items" component={Items} />
              <Route exact path="/form/:formId" component={Form} />
            </Switch>
          </ResponsiveDrawer>
    );
}

```

Take a look at the /form/:formId path above. The formId is a parameter, and it will be used to pick up the right form from the Forms.json. 

### Items.js

After the form is submitted, the control is routed to the Items component. This is the only component to display the result for all APIs. 

```
import React from 'react';
import { useApiGet } from '../hooks/useApiGet';
import CircularProgress from '@material-ui/core/CircularProgress';

function Items(props) {
  console.log(props.location.state.path);
  console.log(props.location.state.accountId);
  console.log(props.location.state.statementId);

  const path = props.location.state.path;
  const accountId = props.location.state.accountId;
  const statementId = props.location.state.statementId;

  const url = statementId ? 'https://ob.lightapi.net' + path + accountId + '/' + statementId : 'https://ob.lightapi.net' + path + accountId
  console.log(url);
  const headers = {
    //Authorization: 'Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ',
    'x-fapi-financial-id': 'OB'
  };

  const { isLoading, data, error } = useApiGet({url, headers});

  let wait;
  if(isLoading) {
    wait = <div><CircularProgress/></div>;
  } else {
    wait = <div><pre>{ JSON.stringify(data, null, 2) }</pre></div>;
  }

  return (
    <div className="App">
      {wait}
    </div>
  );
}

export default Items;

```

Uncomment the Authorization header to use the long-lived token. 



[react-schema-form]: https://github.com/networknt/react-schema-form

