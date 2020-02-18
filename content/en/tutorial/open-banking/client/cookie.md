---
title: "Cookie"
date: 2020-02-09T00:19:45-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have walked through the react-client app with hard-coded [long-lived token][] without hooking up with the light-oauth2 services. 

Before we enable the default chain in the router handler.yml file to add the StatelessAuthHandler to the chain, we are using the hard-coded OAuth 2.0 access token in the useApiGet hook. We can use the react yarn start and test from the localhost:3000 with the hot reload without logging in. Once we switch to the real JWT token with cookies, we cannot do that anymore as the browser doesn't allow the secure cookies to be written if the origin is not the same. We have to use yarn build in the react-client and copy the build folder into the 
`~/networknt/light-config-test/light-router/local-openbanking/ob`. Then we can start the light-router instance to serve the virtual host. 

To test or access the open-banking app, you must use the subdomain https://ob.lightapi.net

After the site is loaded, you need to click the icon on the upper-right corner to log in. 

There are two users available: 

```
stevehu/123456
ericbroda/123456
```

In the last step, we have completed the fetch from the APIs with a long-lived JWT token hardcoded in the useApiGet. In this section, we are going to enable the cookie access from the React app to get the CSRF token and the userId. 

After logging in, you can access the menu to get a list of entities for the logged-in user or get a particular entity associated with a particular account number.

In order to do that, we first need to install the react-cookie. 

```
yarn add react-cookie
```

The above command will add the following dependency to the pacakge.json

```
"react-cookie": "^4.0.3"
```

Next, we need to wrap the App with CookieProvider in the index.js

```
import { CookiesProvider } from 'react-cookie';
.
.
.
            <CookiesProvider>
                <App/>
            </CookiesProvider> 
```

This will allow all function components to use the hook to access cookies. 

After we switch to the StatelessAuth middleware handler, we don't need to use the hardcoded JWT tokens in the function components anymore. Here is the modified Accounts.js with Authorization header commented out. 

```
  const url = 'https://ob.lightapi.net/accounts';
  const headers = {
    //Authorization: 'Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTg5MTcwNDgyNiwianRpIjoiUWttZHRFeE53dDNqemlGSlBtWmFQQSIsImlhdCI6MTU3NjM0NDgyNiwibmJmIjoxNTc2MzQ0NzA2LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlaHUiLCJ1c2VyX3R5cGUiOiJDVVNUT01FUiIsImNsaWVudF9pZCI6ImY3ZDQyMzQ4LWM2NDctNGVmYi1hNTJkLTRjNTc4NzQyMWU3MiIsInNjb3BlIjpbImFjY291bnRzIl19.nqtuQSeeiltEWjXWrdzNrRkKtnqxlO7SUhCMVKzf9zRC0QU4SbdUR99Vbl4MiiTAQR0MxkE5s-BS7KONIyeD4Z2j__6MlcAwx3jCv65HQMCnE8_yMZ5Ut1IoVbNdwcLpDQzWZKbAUpgoUrtw9l_y7zPcyFIHIn0pxo8IiE84ctgfRa1lVU6yjQ8YuTwk5lJmojUToJNeRqXGx73xslrTlXXqF7lLEcCe52cJjbl1oTwzhXIOFllQ85sjbRHWILHpqOKBgpDoQgLqj6Q6aTShlgIjVifbeCZiECamGDUwjXcvFK1mPYy7DWo0PuLJZ0Hy6KaLMP9yr-mpBOSW8Za1pQ',
    'x-fapi-financial-id': 'OB'
  };

```

In fact, the line needs to be removed. But to comment it out at this moment is clearer. 

There are two places that we need to use cookies. The first place is hook useApiGet.

```
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

```

Above, you can see that we get the csrf token from the cookies and put it into the X-CSRF-TOKEN header for each fetch action. It will make sure that we have the CSRF token in each request header so that the StatelessAuth middleware handler can verify it. 

The other place that uses cookie is the Form function component. In the form, we need to prepopulate the userId in order to enable conditional display the dropdown values based on the userId in the [react-schema-form](https://github.com/networknt/react-schema-form). 


In the Form.js

```
import { useCookies } from 'react-cookie';
.
.
.
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

As you can see, we get the userId from the cookie and then merge it with the formData.model in the setModel in the useEffect. This userId will be used to drive the dropdown list. 


When the StatelessAuthHandler is invoked, it will write the access token, refresh token into the HttpOnly, Secure Cookie and the Javascript application cannot access; however, we must ask the browser to send the cookie back to the server so that the server can get the access token, verify the expiration and put into the request header before routing it to the APIs. With fetch API, you need to add additional parameters to the request. The following is the `credentials` values and we have to use `include` as we are redirect from the site to signin.lightapi.net and then redirect back. 

```
A RequestCredentials dictionary value indicating whether the user agent should send cookies from the other domain in the case of cross-origin requests. Possible values are:

omit: Never send or receive cookies.
same-origin: Send user credentials (cookies, basic http auth, etc..) if the URL is on the same origin as the calling script. This is the default value.
include: Always send user credentials (cookies, basic http auth, etc..), even for cross-origin calls.
This is similar to XHR’s withCredentials flag, but with three available values instead of two.
```

Here is the fetch in the useApiGet hook.

```
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

```

We also need to set the same for the siginin.lightapi.net for the original login request. Here is the section in the light-oauth2/login-view/src/App.js. 


```
    fetch("/oauth2/code", {
      method: 'POST',
      redirect: 'follow',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: formData
    })
    .then(response => {
      // HTTP redirect.
      if (response.ok && response.redirected) {
        window.location.href = response.url;
      } else {
        throw Error(response.statusText);
      }
    })
    .catch(error => {
        console.log("error=", error);
        setError(error.toString());
    });

```

This completes the react-client for the Open Banking demo application for the OAuth 2.0 Authorization Code flow. In the next step, we are going to walk though the [login-view][] from the light-oauth2. 


[long-lived token]: /tutorial/open-banking/client/long-live/
[login-view]: /tutorial/open-banking/client/login-view/

