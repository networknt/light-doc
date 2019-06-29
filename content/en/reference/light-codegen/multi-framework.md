---
title: "Multi Framework"
date: 2019-01-05T11:16:49-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Whether or not you are using the command line or the website to generate code, you can choose more than one frameworks at a time to combine the framework code together. Normally, you choose one to generate backend service and another one to generate front-end single page application based on Angular or React. 

For command line tool, you can choose to generate the backend service first to a target directory and then run another command line to generate the front end application into the same target folder. The final target folder should have a running application with both front end and back end tightly integrated together.

For the web interface, it is a little bit complicated as both backend and frontend frameworks must be selected before triggering generation and the final result is zipped and moved to the download folder. You first choose the first framework from a dropdown and give model and config (can be an URL link or upload/copy from the local file system in JSON/YAML/IDL format). And then you can choose the second framework and provide a detailed model and config. Once all frameworks are selected and configured, you click the submit button to send the request to the server. The server response has an URL to the downloadable zip file that contains all the generated files. You just need to click the link to download the project file to your local drive. Then unzip, build, execute and test your project. 

