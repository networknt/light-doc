---
title: "User Management"
date: 2017-11-10T14:51:17-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


The OAuth2 services can be integrated with existing Active Directory, LDAP or customer
database for authentication. If there is no existing authentication service, you can
register users into database as default implementation. For the light-portal we are using
our own database for all users. 

In this tutorial, we are using curl commmand to access the API and endpoints. In reality
these endpoints will only be consumed by light-portal UI.

To add a new user.

```
curl -k -H "Content-Type: application/json" -X POST -d '{"userId":"stevehu","userType":"employee","firstName":"Steve","lastName":"Hu","email":"stevehu@gmail.com","password":"123456","passwordConfirm":"123456"}' https://localhost:6885/oauth2/user
```

To query a user.

```
curl -k https://localhost:6885/oauth2/user/stevehu

```

And here is the result.

```
{"firstName":"Steve","lastName":"Hu","userType":"employee","userId":"stevehu","email":"stevehu@gmail.com"}
```

To update the user type to partner.

```
curl -k -H "Content-Type: application/json" -X PUT -d '{"firstName":"Steve","lastName":"Hu","userType":"partner","userId":"stevehu","email":"stevehu@gmail.com"}' https://localhost:6885/oauth2/user
```

To reset the password.

```
curl -k -H "Content-Type: application/json" -X POST -d '{"password":"123456","newPassword":"stevehu","newPasswordConfirm":"stevehu"}' https://localhost:6885/oauth2/password/stevehu
```

To remove a user.

```
curl -k -X DELETE https://localhost:6885/oauth2/user/stevehu

```
