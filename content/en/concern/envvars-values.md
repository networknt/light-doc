---
title: "Env vars injected into values.yml"
date: 2024-10-17T16:08:19-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

Most light-4j components has config files and the property values can be injected via values.yml file or environment variables. Recently, one of our customers want to inject values into values.yml from environment variables. This is due to the sensitive information is managed by Hashicorp product that can updates the environment variables automatically without checking into git repository in values.yml file. 





## Injecting Environment Variables into values.yml for Light-4j Components

### Current Situation

Light-4j components use config files
Property values can be injected via values.yml file or environment variables
A customer wants to inject values into values.yml from environment variables
This is to leverage Hashicorp products for managing sensitive information without checking it into git

### Proposed Solution

##### Environment Variable Placeholder Syntax

* Introduce a special syntax in values.yml to indicate where environment variables should be injected
* Example: ${ENV_VAR_NAME}


##### Configuration Loader Enhancement

* Modify the configuration loader to recognize and process these placeholders
* When loading values.yml, replace placeholders with actual environment variable values


##### Fallback Mechanism

* Implement a fallback mechanism for when environment variables are not set
* Example: ${ENV_VAR_NAME:default_value}


##### Nested Property Support

* Ensure the solution works for nested properties in YAML


##### Security Considerations

* Implement proper error handling to avoid exposing sensitive information in logs or error messages


### Benefits

* Increased security for sensitive information
* Easier integration with secret management tools like Hashicorp Vault
* More flexible configuration management
* Separation of configuration from sensitive data

### Implementation Steps

* Update the configuration loading mechanism in Light-4j
* Allow the ConfigInjection class to inject values.yml
* Remove the values from the exclusionFileList from config.yml

### Considerations

* Backward compatibility with existing values.yml files is guaranteed. 
* Performance impact of additional processing during configuration loading is negligible.

### Overwrite Order

* Environment variables > values.yml > other config files. 

### GitHub

* PR for the implementation can be found at https://github.com/networknt/light-4j/pull/2350
* Test configuration for the light-gateway can be found at https://github.com/networknt/light-gateway/tree/master/config/values-envvar

### Test

We are using light-gateway as the testing instance with the following configuration. 

values.yml

```
router.hostWhitelist:
  - 192.168.0.*
  - localhost
  - ${DOCKER_HOST_IP}

```

As you can see that we have three white list items in the router.hostWhiteList and the last one is from an environment variable. In my .profile file in my home directory, I have the following line. 


```
export DOCKER_HOST_IP=192.168.5.10
```

Once the server is up and running, you can the third item is replaced with 192.168.5.10 in the hostWhiteList. 

