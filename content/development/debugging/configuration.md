---
title: "Alternative Configuration"
date: 2018-09-01T09:23:26-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Most of our users create unit or integration test cases in the src/test folder to ensure that the code in the src/main folder works as expected. The unit and integration test cases also help to ensure that new changes won't break the existing codebase. The other benefit for these test cases is to be an entry point for the debugger. You can set the breakpoints in your handlers or any middleware handlers and start a debug session with one of the test cases to start the server and place the right request to trigger the logic in your handlers. 

This is all good until you want to test different configuration file combinations. All in all, the flexibility of the light platform is that all components are configurable to change the runtime behavior. In the src/test/resources/config folder, you can only specify one set of config files. You can change them and test different configurations, but you cannot keep all sets of config files in the source control system. 

Since the light-4j supports externalized config files, you can put other alternative config files into another folder in your project and specify the folder in the VM options before starting the debug session. For me, I am using [light-config-test][] repository for most of the alternative debug configurations, and you can find a lot of examples there. 

Here is option to overwrite config folder in the VM options of the IDEA. 

```
-Dlight-4j-config-dir=/home/steve/networknt/light-config-test/xxx/config
```

[light-config-test]: https://github.com/networknt/light-config-test
