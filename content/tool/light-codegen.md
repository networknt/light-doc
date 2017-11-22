---
title: "Light Codegen Tool"
date: 2017-11-08T11:39:11-05:00
description: "Command line tool that is used to scaffold project for light-4j frameworks"
categories: [tool]
keywords: []
menu:
  docs:
    parent: "tool"
    weight: 80
weight: 80	#rem
aliases: []
toc: false
draft: false
---

### Why this generator

In the beginning, we only have Restful framework and we are leveraging [swagger-codegen][]
to generator our projects from Swagger 2.0 specifications. However, there are several issues
that we cannot resolve. 

- swagger-codegen doesn't support Java 8 so we have to fork it and customize it.
- swagger-codegen doesn't give your flexibility to customize the project. i.e. if oracle is supported etc.
- swagger-codegen can only support swagger specification and we later added GraphQL and Hybrid frameworks
- swagger-codegen is very slow and it takes several seconds to generate a project. light-codegen needs several milli-seconds so that we can support code generation from web.
- swagger-codegen doesn't support external extension, means you have to add your generator to the codebase in order to work. 
 
Given all the drawbacks, we have decided to build our own generator. light-codegen is built
with Java 8 and using rocker template engine which compiles the templates to Java class to speed
it up. It also use our IoC service module to enable new generator to be added without touch the
source code of light-codegen. 


### Usage

Light-codegen supports all the frameworks we build and it can be easily extended to other
frameworks. For usage of out-of-box generators, please refer to [generator tutorials][]

### Deploy

Most users will use the generator in Java command line, docker command line or script
which can be easily integrated with any DevOp pipeline.

For users who don't want to clone and build the generator locally, we provide a web
service with UI so that you can generate project remotely and then download/build
locally. 

codegen-web is the web service built to handle request from the Internet. The UI is 
built in React to help user to choose the right parameters and model to control how
generator works. This web service is built on top of light-hybrid-4j framework and
it cannot be started on its own. In order to start the web service, we must first
generate a hybrid server and then put the codegen-web jar file into the classpath
of the server.

Except Web UI is working in progress, command line, docker and docker with script are
working stable at the moment. 


### Customize

light-codegen is a generic code generator based [Rocker template engine]. It can be
extended to support other frameworks or other languages. You just need to package
the new generator along with template files into a jar and put the jar file into the
class path of light-codegen to get it work. In order for the light-codegen to find
your new generator, you need to externalize service.yml to add your generator impl
into it. The one used internally can be found at https://github.com/networknt/light-codegen/blob/master/codegen-fwk/src/main/resources/config/service.yml

### Multiple Frameworks

Whether or not you are using command line or the web site to generate code, you can choose more
than one frameworks at a time to combine the framework code together. Normally, you choose one to
generate backend service and another one to generate front end single page application based on
Angular or React. 

For command line tool, you can choose to generate the backend service first to a target directory
and then run another command line to generate the front end application into the same target folder.
The final target folder should have a running application with both front end and back end tightly
integrated together.

For web interface, it is a little bit complicated as both backend and frontend frameworks must
be selected before triggering generation and the final result will be zipped and moved to download
folder. You first choose the first framework from a dropdown and give model and config (can be a
url link or upload/copy from local file system in JSON format). And then you can choose the second
framework and provide detailed model and config. After all frameworks are selected and configured,
you click the submit button to send the request to the server. The server response will have a
url to the downloadable zip file that contains all the generated files. You just need to click the
link to download the project file to your local drive. Then unzip, build, execute and test your
project. 


[swagger-codegen]: https://github.com/swagger-api/swagger-codegen
[generator tutorials]: /tutorial/generator/
[Rocker template engine]: https://github.com/fizzed/rocker
