---
title: "Custom Runtime"
date: 2021-01-15T17:25:01-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

This is the custom runtime that is used to deploy the lambda function natively with GraalVM. It is a component in the [light-aws-lambda][] repository. 

The Runtime class is the entry point with the main method. Each lambda function will specify this class as the Main-Class in the manifest. 

```
jar {
  manifest {
    attributes 'Main-Class': 'com.networknt.aws.lambda.Runtime'
  }
}
```

[light-aws-lambda]: https://github.com/networknt/light-aws-lambda/tree/master/custom-runtime

