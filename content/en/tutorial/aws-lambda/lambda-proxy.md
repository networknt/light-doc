---
title: "Lambda Proxy"
date: 2021-01-20T12:49:52-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

In this tutorial, we are going to use the light-proxy to address the cross-cutting concerns. We are going to scaffold a project with four Lambda functions based on the petstore OpenAPI 3.0 specification. 

### Specification

The petstore specification [openapi.yaml][] and a [config-proxy.json][] can be found in [model-config][] repository in the networknt GitHub organization. 

This folder also contains a [README.md][] with the commmand lines to generate the petstore project. To generate a project with light-proxy, we need to use the last command in the README.md

To generate the project with local reference, we need to check out this repo. 

```
cd ~/networknt
git clone https://github.com/networknt/model-config
```

### Light-codegen

We are going to use the light-codegen to generate the project. To make sure we use the latest code, we need to switch to the master branch from the default release branch. 

```
cd ~/networknt
git clone git@github.com:networknt/light-codegen.git
cd light-codegen 
git checkout master
mvn clean install
```

### Light-example-4j

With the light-codegen is built, we will generate a new project into the light-example-4j project. If you don't want to regenerate the project, you can use the generated one in the light-example-4j lambda [petstore-proxy][] folder. Check out the repository and switch to the master branch. 

```
cd ~/networknt
git clone git@github.com:networknt/light-example-4j.git
cd light-exmaple-4j
git checkout master
```

If you want to regenerate the project petstore-proxy, run the following command lines. Make sure you have the light-example-4j project checked out into the `~networknt` folder before running the commands.

```
cd ~/networknt
rm -rf light-example-4j/lambda/petstore-proxy
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f openapilambda -o light-example-4j/lambda/petstore-proxy -m model-config/lambda/petstore/openapi.yaml -c model-config/lambda/petstore/config-proxy.json
```

### README.md

In the generated petstore-proxy project, a README.md describes all the components and command lines to build and deploy these components. 


### Proxy Docker

To demonstrate the JWT verification, we want to update the open-security.yml to set enableVerifyJwt to true. 

```
enableVerifyJwt: ${openapi-security.enableVerifyJwt:true}
```

To build and push the light-proxy image to the ECR. We run the following command. Before you do that, you need to set up the aws cli. 

```
cd ~/networknt/light-example-4j/lambda/petstore-proxy
chmod +x build.sh
./build.sh 1.0.0
```

### Deploy VPC

To deploy the light-proxy containers, we need to create the VPC with 2 EC2 instances and two public subnets.

```
aws cloudformation create-stack --stack-name petstore-vpc --template-body file://public-vpc.yaml --capabilities CAPABILITY_IAM
```

### Deploy Lambda Functions

Build Lambda functions

```
sam build
```

Deploy Lambda functions to the us-east-2


```
sam deploy --guided
```

### Deploy Light-Proxy

Use the following command to deploy the light-proxy. You need to create an accessKeyId and a secretAccessKey. 

```
aws cloudformation create-stack \
  --stack-name petstore-proxy \
  --template-body file://public-proxy.yaml \
  --parameters \
      ParameterKey=StackName,ParameterValue=petstore-vpc \
      ParameterKey=ServiceName,ParameterValue=lambda-proxy \
      ParameterKey=ImageUrl,ParameterValue=964637446810.dkr.ecr.us-east-2.amazonaws.com/lambda-proxy:latest \
      ParameterKey=ContainerPort,ParameterValue=8080 \
      ParameterKey=HealthCheckPath,ParameterValue=/health/com.networknt.petstore-3.0.1 \
      ParameterKey=HealthCheckIntervalSeconds,ParameterValue=90 \
      ParameterKey=AccessKeyId,ParameterValue=YourKeyId \
      ParameterKey=SecretAccessKey,ParameterValue=YourSecret
```

Replace the ImageUrl, AccessKeyId and SecretAccessKey in the above command.

### Test

Once all stacks are deployed, we can send a request to the external url. The external url can be found from the following command from the petstore-vpc stack. 

```
aws cloudformation describe-stacks
```

With the url, we can send a curl command.

```
http://petst-publi-1hvkn942lczba-1854182859.us-east-2.elb.amazonaws.com/v1/pets
```

You will get the following error. 

```
{"statusCode":401,"code":"ERR10002","message":"MISSING_AUTH_TOKEN","description":"No Authorization header or the token is not bearer type","severity":"ERROR"}
```

Now, let's add a long-lived token to the header.


```
curl -X GET http://petst-publi-1hvkn942lczba-1854182859.us-east-2.elb.amazonaws.com/v1/pets \
-H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTc5MDAzNTcwOSwianRpIjoiSTJnSmdBSHN6NzJEV2JWdUFMdUU2QSIsImlhdCI6MTQ3NDY3NTcwOSwibmJmIjoxNDc0Njc1NTg5LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ3cml0ZTpwZXRzIiwicmVhZDpwZXRzIl19.mue6eh70kGS3Nt2BCYz7ViqwO7lh_4JSFwcHYdJMY6VfgKTHhsIGKq2uEDt3zwT56JFAePwAxENMGUTGvgceVneQzyfQsJeVGbqw55E9IfM_uSM-YcHwTfR7eSLExN4pbqzVDI353sSOvXxA98ZtJlUZKgXNE1Ngun3XFORCRIB_eH8B0FY_nT_D1Dq2WJrR-re-fbR6_va95vwoUdCofLRa4IpDfXXx19ZlAtfiVO44nw6CS8O87eGfAm7rCMZIzkWlCOFWjNHnCeRsh7CVdEH34LF-B48beiG5lM7h4N12-EME8_VDefgMjZ8eqs1ICvJMxdIut58oYbdnkwTjkA' 
```

This time you will have the result.

[model-config]: https://github.com/networknt/model-config/tree/master/lambda/petstore
[openapi.yaml]: https://github.com/networknt/model-config/blob/master/lambda/petstore/openapi.yaml
[config-proxy.json]: https://github.com/networknt/model-config/blob/master/lambda/petstore/config-proxy.json
[README.md]: https://github.com/networknt/model-config/tree/master/lambda/petstore
[petstore-proxy]: https://github.com/networknt/light-example-4j/tree/master/lambda/petstore-proxy

