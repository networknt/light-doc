---
title: "Multiple Rule Actions"
date: 2024-11-19T11:00:44-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

So far, we know how to use the yaml-rule-plugin to perform the [request & response transformation][] in the chain for most of the use cases. However, if there are complex senarios that need more transform steps or plugins, we can configure the light-gateway values.yml and rules.yml to chain multiple rule actions together. 





Here is the log in openapi-petstore terminal. 

```
2024-11-19T10:33:37.862 [XNIO-1 task-2]  test-correlation INFO  c.n.p.handler.FlowersPostHandler:38 handleRequest - requestHeaders = {accept-encoding=[gzip, deflate, br], x-traceability-id=[test-traceability], x-forwarded-port=[8443], x-forwarded-for=[0:0:0:0:0:0:0:1], vnd.com.blackrock.api-key=[test-api-key], x-forwarded-host=[localhost], Host=[localhost:9443], accept=[*/*], postman-token=[ab682d5a-59bc-448c-a71a-598f91e8c34b], vnd.com.blackrock.request-id=[test-request-id], x-correlation-id=[test-correlation], x-disable-push=[true], x-forwarded-server=[localhost], user-agent=[PostmanRuntime/7.29.4], my-header=[test1], authorization=[Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5NDg3MzA1MiwianRpIjoiSjFKdmR1bFFRMUF6cjhTNlJueHEwQSIsImlhdCI6MTQ3OTUxMzA1MiwibmJmIjoxNDc5NTEyOTMyLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.gUcM-JxNBH7rtoRUlxmaK6P4xZdEOueEqIBNddAAx4SyWSy2sV7d7MjAog6k7bInXzV0PWOZZ-JdgTTSn6jTb4K3Je49BcGz1BRwzTslJIOwmvqyziF3lcg6aF5iWOTjmVEF0zXwMJtMc_IcF9FAA8iQi2s5l0DYgkMrjkQ3fBhWnopgfkzjbCuZU2mHDSQ6DJmomWpnE9hDxBp_lGjsQ73HWNNKN-XmBEzH-nz-K5-2wm_hiCq3d0BXm57VxuL7dxpnIwhOIMAYR04PvYHyz2S-Nu9dw6apenfyKe8-ydVt7KHnnWWmk1ErlFzCHmsDigJz0ku0QX59lM7xY5i4qA], x-test-1=[test1], x-forwarded-proto=[https], your-header=[test1], content-type=[text/xml], content-length=[1173]}
2024-11-19T10:33:37.863 [XNIO-1 task-2]  test-correlation INFO  c.n.p.handler.FlowersPostHandler:41 handleRequest - body = <SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header>   <wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd"
               xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd"
               soapenv:mustUnderstand="1">
      <wsse:UsernameToken wsu:Id="UsernameToken-eab4be46-8374-4492-81c9-502798f4b123">
         <wsse:Username>SLUATWS</wsse:Username>
         <wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordDigest">X7wWUEAWb5ofllrk4IZgLM8UPO4=</wsse:Password>
         <wsse:Nonce EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">MTczMjAzMDQxNzgwMA==</wsse:Nonce>
         <wsu:Created>2024-11-19T15:33:37.801Z</wsu:Created>
      </wsse:UsernameToken>
   </wsse:Security>
</soapenv:Header>
  <SOAP-ENV:Body>
    <ea:Flower xmlns:ea="http://www.w3.org/2001/XMLSchema">
      <name>Poppy</name>
      <color>RED</color>
      <petals>9</petals>
    </ea:Flower>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>

```
The above 

[request & response transformation]: /tutorial/gateway/request-response-transformation/
