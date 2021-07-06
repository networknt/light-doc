---
title: "How to generate long lived tokens"
date: 2017-11-10T14:50:02-05:00
description: ""
categories: []
keywords: []
weight: 10
aliases: []
toc: false
draft: false
reviewed: true
---

Normally, your clients or testing tools should integrate with OAuth authorization server to get tokens during runtime; however, for manual testing, it is very inconvenient to get an access token every 10 minutes. To make the testers' jobs easier, you can generate a long-lived token for dev testing from a tool in the light-4j framework.

The light-oauth2 contains two testing key pairs for testing only. Both private keys and public key certificates can be found in [security module of light-4j][] or [light-docker oauth2-token config][]. The same public key certificates are included in light-4j services generated from [light-codegen][] so that the server can verify any token issued by the OAuth server.

***Important note:***
For your official deployment, please create self-signed key pairs of your own or buy a certificate from one of the Commercial CAs.


The tool is a test case that you can add an entry to generate a token based on your needs. We are considering building a site for developers to get a long-lived token in the future. If a site is provided, it should be part of the portal at https://lightapi.net/form/longLivedTokenForm.

Please be aware that the portal can only generate a token based on the default key pair mentioned above. If you deploy your light-oauth2 instance with your self-signed or CA-issued certificates, then the site won't work. 

It is locate at light-4j/security/src/test/java/com/networknt/security/JwtHelperTest.java

[Here][] is the url on github.com networknt organization. 

Here are two examples that generate authorization code grant type tokens. You can add your own test cases with different parameters.


```
    @Test
    public void longLivedATMP1000Jwt() throws Exception {
        JwtClaims claims = getTestClaims("eric", "EMPLOYEE", "f7d42348-c647-4efb-a52d-4c5787421e72", Arrays.asList("ATMP1000.w", "ATMP1000.r"));
        claims.setExpirationTimeMinutesInTheFuture(5256000);
        String jwt = JwtHelper.getJwt(claims);
        System.out.println("***LongLived ATMP1000 JWT***: " + jwt);
    }


    @Test
    public void longLivedPetStoreJwt() throws Exception {
        JwtClaims claims = getTestClaims("steve", "EMPLOYEE", "f7d42348-c647-4efb-a52d-4c5787421e72", Arrays.asList("write:pets", "read:pets"));
        claims.setExpirationTimeMinutesInTheFuture(5256000);
        String jwt = JwtHelper.getJwt(claims);
        System.out.println("***LongLived PetStore JWT***: " + jwt);
    }

```

The following is a token generated for petstore api with scope write:pets and read:pets

```
Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDgwMDYzOSwianRpIjoiWFhlQmpJYXUwUk5ZSTl3dVF0MWxtUSIsImlhdCI6MTQ3OTQ0MDYzOSwibmJmIjoxNDc5NDQwNTE5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.f5XdkmhOoHT2lgTobqVGPp2aWUv_ItA0tqyLHC_CeMbmwzPvREqb5-oJ9T_m3VwRcJlPTh8xTdSjrLITXClaQFE4Y0bT8C-u6bb38uT-NQ5mjUjLrFQYHCF6GqwL7YkwQt_rshEqtrDFe1T4HoEL_9FHbOxf3MSJ39UKq0Ef_9mHXkn4Y-SHfdapeuUWc_4dDPdxzEdzbqmf1WSOOgTuM5O5F2fK4p_ix8LQl0H3AnMZIhIDyygQEnYPxEG-u35gwh503wfxio6buIf0b2Kku2PXPE36lethZwIVaPTncEcY5OPxfBxXuy-Wq-YQizd7NnpJTteHYbdQXupjK7NDvQ
```

If your light-oauth2 server is using different certificates, then you should copy the certificates and jks files to [this folder][] before running the test case to generate the token. 

[security module of light-4j]: https://github.com/networknt/light-4j/tree/master/security/src/test/resources/config
[light-docker oauth2-token config]: https://github.com/networknt/light-docker/tree/master/light-oauth2/mysql/config/oauth2-token
[light-codegen]: https://github.com/networknt/light-codegen/tree/master/light-rest-4j/src/main/resources/templates/rest
[Here]: https://github.com/networknt/light-4j/blob/master/security/src/test/java/com/networknt/security/JwtHelperTest.java
[this folder]: https://github.com/networknt/light-4j/tree/master/security/src/test/resources/config
