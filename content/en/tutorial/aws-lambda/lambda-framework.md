---
title: "Lambda Framework"
date: 2021-01-20T12:49:42-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

In this tutorial, we are going to use the [light-aws-lambda][] framework to address the cross-cutting concerns. We are going to scaffold a project with four Lambda functions based on the petstore OpenAPI 3.0 specification. 

{{< youtube 7Tz748a2W9I >}}


### Specification

The petstore specification [openapi.yaml][] and a [config-proxy.json][] can be found in [model-config][] repository in the networknt GitHub organization. 

This folder also contains a [README.md][] with the commmand lines to generate the petstore project. To generate a project with light-proxy, we need to use the last command in the README.md

To generate the project with local reference, we need to check out this repo. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config
```

{{< youtube o0gnWu2QLmc >}}


### Codegen


We are going to use the light-codegen to generate the project. To make sure we use the latest code, we need to switch to the master branch from the default release branch. 

```
cd ~/networknt
git clone git@github.com:networknt/light-codegen.git
cd light-codegen 
git checkout master
mvn clean install
```

Run the following commands to generate the project after checkout the model-config repository.

```
cd ~/networknt
rm -rf light-example-4j/lambda/petstore-zip
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapilambda -o light-example-4j/lambda/petstore-zip -m model-config/lambda/petstore/openapi.yaml -c model-config/lambda/petstore/config-zip.json

```

{{< youtube tKUnvBZ8V9Y >}}


### Project

The generated project is located at [light-example-4j](https://github.com/networknt/light-example-4j/tree/release/lambda/petstore-zip)


{{< youtube 2G6DuAp13AM >}}


### Build

Build with sam build command line in the root folder. 

{{< youtube e48iUSxiv18 >}}

### Deploy


Deploy with the sam deploy --guided command line to the aws. 

{{< youtube _RoUjeY_6pk >}}



### Test

From the AWS API Gateway/Stages/Prod, we can find the URLs to access the lambda functions. 

```
curl --location --request GET 'https://jbkvds2jcf.execute-api.us-east-2.amazonaws.com/Prod/v1/pets'
```

You will get the following error. 

```
{"message":"Unauthorized"}
```

Now, let's add a long-lived token to the header.

```
curl --location --request GET 'https://jbkvds2jcf.execute-api.us-east-2.amazonaws.com/Prod/v1/pets' \
--header 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA'
```

This time you will have the result.

{{< youtube 6XaIvKPXNI4 >}}


### Clean up

Clean up all the artifacts deployed.

{{< youtube YrUcDrYsoxs >}}


[light-aws-lambda]: https://github.com/networknt/light-aws-lambda
[model-config]: https://github.com/networknt/model-config/tree/master/lambda/petstore
[openapi.yaml]: https://github.com/networknt/model-config/blob/master/lambda/petstore/openapi.yaml
[config-proxy.json]: https://github.com/networknt/model-config/blob/master/lambda/petstore/config-proxy.json
[README.md]: https://github.com/networknt/model-config/tree/master/lambda/petstore
[petstore-proxy]: https://github.com/networknt/light-example-4j/tree/master/lambda/petstore-proxy
[light-proxy]: https://github.com/networknt/light-proxy


