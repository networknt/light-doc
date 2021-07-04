---
title: "React Light-oauth2 Login View"
date: 2020-02-29T20:17:32-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

For redirect based OAuth 2.0 authorization code flow, a user agent like a web browser or mobile phone needs to be involved. In the light-oauth2 repository, we have provided a [login-view][] single-page application based on React. In this section, we are going to walk through the application to show users the implementation details.

This single-page application should be hosted in the light-router instance as a virtual host along with a customer's application. From the customer's application, a login button will redirect to this login-view application URL with some query parameters like username, password, and client_id, etc.

### Flow

The login-view is based on the redirect Authorization Code flow for Single Page Applications. We will provide another example of the mobile application later.  

* The original application will have a log in button or menu item. Once it is clicked, the browser will redirect to the login-view application, which is based on the React.

* The login-view will accept username and password to invoke light-oauth2 code service. 

* Once authenticated, the code service will redirect to the /authorization?code=xxx to the browser. 

* The browser will invoke the above API, which is handled by StatelessAuthHandler in light-spa-4j. This handler is part of the middleware handler chain on the light-router.

* The StatelessAuthHandler will use the code query parameter, client_id, client_secret saved on router server to get access token and refresh token from the light-oauth2 token service. The StatelessAuthHandler will save the access token, refresh token, csrf, userId, roles, and userType as cookies. The access token and refresh token are HttpOnly and Secure, so the browser application cannot access them. The handler also returns the redirectUri, denyUri, and scopes. 

* The login-view will use the response to display a consent page with a list of scopes and ask users to DENY or ACCEPT the grant. 

* If the DENY button is clicked, all the cookies will be removed, and the browser will be redirected to the denyUri. The entire flow is canceled. 

* If the ACCEPT button is clicked, the browser will be redirected to the redirectUri. 

* All subsequent calls to the light-router server from the original SPA will carry all the cookies to the request. The StatelessAuthHandler will capture the access token and refresh token and put the access token into the request header to invoke downstream APIs. 

* If the access token is expired or about to expire, The StatelessAuthHandler will renew a new token and put the new one into the request header. 

* To prevent CSRF(Cross-site request forgery), the original application must get the csrf cookies and put it into the request header for each request. 

The above is a simplified flow without a lot of detail for easy to understand. 

### Source Code

The source is generated with create-react-app, and the main logic is in the Login.js file. The component accepts query parameters from the redirect and uses the useState for the state, clientId, userType, and redirectUri. 

```
const [state] = useState(params.get('state') == null ? '' : params.get('state'));
```

It renders a login form that accepts username, password, and rememberMe. Once the submit button is clicked, the component will fetch the /oauth2/code to start the Authorization Code flow. If everything is OK, the light-oauth2 code service will redirect to the redirectUri registered in the client registration. In most of the cases, it will be your application domain with https protocol and the path /authorization.

There are two other components: ForgetPassword and ResetPassword. On the login page, there is a link to the forgot password page if you forget your password and want to reset it. You can input your email address on the page and click the submit button to have an email sent to you. In the email, there is a button that will direct you to the reset password page with a hidden token. You can enter your new password and password confirmation to reset your password on this page. 

The login-view is calling the light-oauth2 code service for authentication and authorization. Internally, it uses the Java security GSSAPI to authenticate against the light-portal hybrid-query service. Due to the limitation of the API, there is no way that the error message can be passed to the authentication layer. If authentication is failed, a link will be displayed in the error message. By clicking the link, the hybrid-query loginUser service will be invoked directly, and you can see the error message on the browser. 
 
### Build

Before using the SPA, you need to build the application and deployed as a virtual host on an instance of light-router. There are three different build options depending on where the light-oauth2 services are located. 

* If you deploy the light-oauth2 services on your own behind the light-router, then you can use the following command to build. 

```
yarn build
```

* If you are trying to use the dev light-oauth2 instances provided by us at https://dev.oauth.lightapi.net, then use the following command to build.

```
yarn build-dev
```

* If you are going to deploy to your production and want to use the SaaS production at https://oauth.lightapi.net, then use the following command to build.

```
yarn build-prd
```

When using the dev or production SaaS light-oauth2 instances, the URL will be injected in the final code in the build folder. Here is the logic in the Login.js


```
    const headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        ...(process.env.REACT_APP_SAAS_URL) && {'service_url': process.env.REACT_APP_SAAS_URL}
    };
    console.log(headers);

    fetch("/oauth2/code", {
      method: 'POST',
      redirect: 'follow',
      credentials: 'include',
      headers: headers,
      body: formData
    })
    .then(response => {
      if (response.ok) {
        return response.json();
      } else {
        throw Error(response.statusText);
      }
    })
    .then(json => {
      console.log(json);
      setRedirectUrl(json.redirectUri);
      setDenyUrl(json.denyUri);
      setScopes(json.scopes);
    })
    .catch(error => {
        console.log("error=", error);
        setError(error.toString());
    });

```

And here is the build script in the package.json file.

```
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build-dev": "REACT_APP_SAAS_URL=https://dev.oauth.lightapi.net react-scripts build",
    "build-prd": "REACT_APP_SAAS_URL=https://oauth.lightapi.net react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },

```

For information on how to deploy the https://dev.oauth.lightapi.net, please find the [test-cloud](/tutorial/oauth/test-cloud/) tutorial.

Here is a video walkthrough of the authorization code flow and the login-view single page application. 

{{< youtube VdWqymAKslg >}}

[login-view]: https://github.com/networknt/light-oauth2/tree/master/login-view

