---
title: "Light Reference"
date: 2020-07-09T13:31:42-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

Reference API is a microservice designed to populate UI dropdown lists in real-time with relationship support. Try the link for a demo with react-schema-form.

https://maproot.net/#/app/form/covidMapForm

In the above form, you have to select a country before the province is populated from the reference API. Once a province is select, the cities in the province will be populated in real-time. The category and subcategory have a similar relationship. With the high-performance implementation, we can support loading the reference data from the browser when users are updating one field in the form. The response time of the API is within 10 million seconds under thousands of concurrent requests. 

For any organization, there is a lot of reference information that should be centrally managed and shared by different applications. The dropdown population is only part of the usage, and some of the reference data might be used in the business logic to make decisions in the flow control. 

Along with light-reference, there are ref-command and ref-query in the light-portal to update the reference data and relationship. 

If you are interested in deploy this service in your data center or sign up our hosting service, please contact sales@lightapi.net for details. 
