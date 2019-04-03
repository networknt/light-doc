---
title: "Graphql Validator"
date: 2017-11-29T20:59:38-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Graphql Validator validates the path and methods of the request. Other schema validation will be handled by the GraphQL component.

Basic request validation for the graphql path and methods is done by this middleware handler. It is the first line of validation right after graphql-security and it doesn’t have any knowledge about the graphql query parameter and body. Other schema based validation will be done at GraphQL level.

From release 1.5.18, the light platform supports multiple chains of middleware handlers and multiple frameworks mixed in the same service instance. To have a validator configuration file for different frameworks, a new graphql-validator.yml with the same content has been introduced. The validator.yml is still loaded if graphql-validator.yml doesn't exist for backward compatibility. 

Here is an example of graphql-validator.yml


```
# This is specific GraphQL validator configuration file. It is introduced to support multiple
# frameworks in the same server instance and it is recommended. If this file cannot be found,
# the generic validator.yml will be loaded as a fallback.
---
# Enable request validation against the specification
enabled: true
# Log error message if validation error occurs
logError: true
```
