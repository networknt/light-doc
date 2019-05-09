---
title: "SPA Session JWT"
date: 2018-05-01T09:38:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

When building Single Page Applications that consume APIs protected by JWT tokens or simple web token, a lot of articles suggest to put the token into local storage on the browser so that the Javascript application can use the token to access APIs directly. **Please don't do that as it is not safe**. 

As the article [stop using localstorage] states, local storage on the browser is not security. It is subject to cross-site scripting attack, and the information can be retrieved by anyone who has access to the browser physically. 

If you need to store sensitive data, you should always use a server-side session. Here's how to do it:

* When a user logs into your website, create a session identifier for them and store it in a cryptographically signed cookie. If you're using a web framework, look up “how to create a user session using cookies” and follow that guide.

* Make sure that whatever cookie library your web framework uses is setting the httpOnly cookie flag. This flag makes it impossible for a browser to read any cookies, which is required in order to safely use server-side sessions with cookies. Read [Jeff Atwood's article][] for more information. He's the man.

* Make sure that your cookie library also sets the SameSite=strict cookie flag (to prevent [CSRF][] attacks), as well as the secure=true flag (to ensure cookies can only be set over an encrypted connection).

* Each time a user makes a request to your site, use their session ID (extracted from the cookie they send to you) to retrieve their account details from either a database or a cache (depending on how large your website is)

* Once you have the user's account info pulled up and verified, feel free to pull any associated sensitive data along with it

This pattern is simple, straightforward, and most importantly: secure. And yes, you can most definitely scale up a large website using this pattern when using [light-session-4j][] which is a distributed session backed by in-memory data grid. 


[stop using localstorage]: https://dev.to/rdegges/please-stop-using-local-storage-1i04
[Jeff Atwood's article]: https://blog.codinghorror.com/protecting-your-cookies-httponly/
[CSRF]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)
[light-session-4j]: /sytle/light-session-4j/

