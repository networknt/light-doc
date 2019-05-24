---
title: "Quick Start - Microservices Chain Application"
date: 2017-11-29T10:12:34-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
toc: true
---
You will need the following dependencies before starting this tutorial:

- Java JDK 8 (I prefer OpenJDK but Oracle JDK will do)
- Maven
- Git
- Docker

This quick start tutorial is designed for anyone beginning to utilize Light-4j and wants to gain a general understanding of how to setup a microservice chain application utilizing four APIs. This example found in *[light-example-4j/rest/swagger/ms_chain](https://github.com/networknt/light-example-4j)* shows four APIs being utilized as such:
```
API A -> API B, API B -> API C, API C -> API D
```
## Setting up the Environment
For the purpose of this guide we will be using IntelliJ's Java IDE.

After cloning the above git repository, the *ms_chain* directory must be opened and in order to correctly run Maven, we must go into the directory containing the **pom.xml** file *(/rest/swagger/api_a/httpschain)* and option click>*"Add as Maven Project"*. This will create a **classpath of module** which should be selected in IntelliJ's *"Edit Configurations"*.

Once we have our environment set up, we can begin to test it and see if API A will call the other three APIs.

## Execution

In order to run the Microservices Chain Pattern we must finally execute all APIs (A, B, C, and D) onto their ports. The example sets up the ports 7441, 7442, 7443, and 7444, respectively. Within the separate API folder directories (E.g. api_a/httpschain) run the Maven command to run the API on the port:

```
mvn install exec:exec
```
>If you do not specify where your pom.xml file is, we must run the maven command from the directory it is in. In this example it is in api_a/httpschain. 
>Also note that our api_a/generated does not include the chain services code, but is simply the "blueprint" of our APIs.

Following many completed tests, the output for running API A should be:
```
Https Server started on ip:0.0.0.0 Port:7441
```
Run this three more times for APIs B, C, and D. The output, excluding port numbers, should be the same.

>If you come across an error when initializing a port, visit your *server.yml* file in each directory (A, B, C, and D) and change the *Httpsport" number to a unique port currently not in use.
>Note: each API must have a separate port number.

Once this is completed separately for all four APIs, you can begin execution and testing. Open a new shell and execute the following command.

```
curl -k https://localhost:7441/v1/data
```
##### What is curl? _Curl simply sends a request. In the command line, the curl command is entered with the option *-k* as it allows curl to proceed and operate even for server connections otherwise considered insecure. Furthermore, we input port 7441 for API A. Our */v1/data* is our *GET Handler* from our **PathHandlerProvider.java**._
The output following this is:

```
["API D: Message 1","API D: Message 2","API C: Message 1","API C: Message 2","API B: Message 1","API B: Message 2","API A: Message 1","API A: Message 2"]%
```
Which if we study our **DataGetHandler.java** (Just from API A) we can see the final output seems to be a returned list of appended strings:

```java
API_A FILE

list.add("API A: Message 1");
list.add("API A: Message 2");
```
The remaining 3 API files also append their strings, and the final output returns the list.

## Explore Further

In order to understand the differences between these four entities, explore the **PathHandlerProvider.java** in the respective directories **(api_x/httpschain)**. API A holds more connection establishment calls and checks than API D.

## Attempt Other Handlers

Other handlers can be found in **PathHandlerProvider.java** and utilized in the same way as our initial *curl* for example:
```
curl -k https://localhost:7441/v1/health
```
and will output:
```
Ok%
```   
&nbsp;


# Conclusion

In this tutorial, a Microservice Chain Pattern application was implemented locally with general understanding of the difference between our *generated* directory and our *httpschain* directory, and their differing uses. It is also recommended to strengthen your understanding by creating your own system using Light-4j

Return to our [Tutorials][] page to explore more.




[Swagger 2.0 specification]: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md
[OpenAPI 3.0 specification]: https://swagger.io/specification/
[Tutorials]: /tutorial