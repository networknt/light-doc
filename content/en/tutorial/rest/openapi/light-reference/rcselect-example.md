---
title: "RcSelect Example"
date: 2019-12-02T00:35:50-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have the reference API [light-router configuration][] done, and the API can be accessed through the lightapi.net domain with HTTPS/HTTP2. In this step, we are going to add a new dynamic-single.json for a form that accesses to the reference API with table name as `role`

First, we will run the example from the local desktop. The port number is 3000, as it is a create-react-app application. 

Add the following file to the react-schema-form-rc-select/example/public/data folder. 


dynamic-single.json

```
{
    "schema": {
        "type": "object",
        "title": "React Component Select Dynamic Single Demo",
        "required": [
            "role"
        ],
        "properties": {
            "name": {
                "title": "name",
                "type": "string"
            },
            "role": {
                "title": "Dynamic Single Select",
                "type": "string"
            }
        }
    },
    "form": [
        "name",
        {
            "key": "role",
            "type": "rc-select",
            "action": {
                "get": {
                    "url" : "https://lightapi.net/rdata/com.networknt?name=role"
                }
            }
        }
    ]
}

```

Please note that the URL is specified in the form.action.get to point to the reference API. 

In the App.js, add the following line to the tests array. 

```
            { label: "Dynamic Single", value: 'data/dynamic-single.json'},
```

On the terminal, run the `npm start` to start the react application. A browser tab will be opened, and let's select the dynamic-single. The dropdown is empty, and the console is displaying an error message. 

```
Access to fetch at 'https://lightapi.net/rdata/com.networknt?name=role' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

This is because the SPA is running on the localhost and the reference API is running on the lightapi.net domain. The lightapi.net rejects the request from the localhost.

Now, let's check the cors.yml on the light-router config folder in the light-config-test repo. 

cors.yml

```
enabled: true
allowedOrigins:
  - https://signin.lightapi.net
  - https://lightapi.net
  - https://dev.lightapi.net
  - https://localhost:3000
allowedMethods:
  - GET
  - POST
  - PUT

```

It does have the localhost:3000 allowed but through https. Let's update it to HTTP. The demo application will be deployed to the networknt.github.io eventually, so we add that domain as well. 

```
enabled: true
allowedOrigins:
  - https://signin.lightapi.net
  - https://lightapi.net
  - https://dev.lightapi.net
  - http://localhost:3000
  - http://networknt.github.io
allowedMethods:
  - GET
  - POST
  - PUT

```

Check the file into the github and go to the portal server.

```
ssh portal
cd networknt/light-config-test
git pull origin master
cd light-router/test-portal
docker-compose down
docker-compose up -d

```

Now, go back to the browser to access the example application running on the localhost:3000 and select Dynamic Single again in the dropdown. The form displaced correctly with both user and admin in the dropdown. 

Let's add another form to the tests with the following json file. 

dyanmic-multiple.json

```
{
    "schema": {
        "type": "object",
        "title": "React Component Select Dynamic Multiple Demo",
        "required": [
            "clients"
        ],
        "properties": {
            "name": {
                "title": "name",
                "type": "string"
            },
            "clients": {
                "title": "Dynamic Multiple Select",
                "type": "array"
            }
        }
    },
    "form": [
        "name",
        {
            "key": "clients",
            "type": "rc-select",
            "multiple": true,
            "action": {
                "get": {
                    "url" : "https://lightapi.net/rdata/com.networknt?name=client"
                }
            }

        }
    ]
}
```

In the next step, we are going to deploy the latest example to the networknt.github.io site by following the [deploy tutorial][]. Wait several minutes and test if the new dynamic forms are there. With no surprise, it works. 

The RcSelect also support post request with parameters, but we don't have post API available for a demo yet. In real production, we will move the single page application to the lightapi.net so that we don't need to bother the CORS configuration. 

This is a long tutorial that shows users how to create a service and deploy it with docker on the VM and config the light-router to access it. It also demos the front-end SPA connectivity to the API through the react-schema-form and the add-on component RcSelect. Although missing some details, it is a good example to show users how to build and deploy a real application to the production with backend light-4j service and front end React with react-schema-form. 


[light-router configuration]: /tutorial/rest/openapi/light-reference/router-config/
[deploy tutorial]: /consumer/react-schema-form-rc-select/developer-guide/#demo-site
