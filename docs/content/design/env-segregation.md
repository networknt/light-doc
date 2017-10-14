# Environment Segregation

There are two goals for environment segregation: 

* Prevent cross fire

For example, UAT test accesses production services or production application accidentally accesses test
database etc.

* Support multiple instances for testing

For example, one service in testing has multiple backend database and different departments will hit their
own instance which connects to their particular database.

The above use cases are pretty complicated even in traditional monolithic application. In microservices
architecture, it is even more complicated as there are two many services involves and each service has
many instances. 


## To prevent cross fire

In [light-4j](https://github.com/networknt/light-4j) framework, we are using [light-oauth2](https://github.com/networknt/light-oauth2)
to separate different environments. For each environment, there is a oauth2 provider to sign JWT token
with its private key. All the services that is deployed to that environment will have the public key from
the provider. So even if there is an UAT service trying to access a production service, the token provided
by the caller cannot be verified due to invalid signature. Given above explanation, there is no way that
we can have cross fire if configuration for each application and service is correct. 

 
So how can we prevent mistakes when deploy services to the cloud environment? How can we make sure that
the configuration files won't be wrong during deployment? In our ecosystem, we encourage automatically
deployment without human involves so that we can eliminate human errors during deployment. Also, we have
light-config-server to serve configuration for each environment and these configuration are in separate
github repos and managed by different teams or people. 

  
## To support multiple instances per environment

One of my customers has an API developed by one line of business but it is shared by others. Each line of
business has its own test database with different set of reference tables to be populated for their own
testing. So for each API release, they have to start several instances which connect to different databases
and give the proper instance to certain QA team to test.

Ideally, the test environment should be consolidated to have the same reference data and similar data set
as production; however, is has been this way for over a decade and it is too costly to update the test data.

The solution we provided is to have an environment variable in server.yml to indicate which backend database
the instance is connecting to. This additional attribute will be registered to consul for service discovery
and the service discovery logic will check the variable to add additional filter if it exists. 

The above logic resolves service to service calls to be bound to the same environment. For the original
client, for example Java EE based web server or test tool like load runner, it must either use our client
module to do the discovery with an additional environment or they have to manually do the lookup on consul
to find the right instance with the right environment tag. 

This solution normally will only be used in testing environment not production. In production, there should
not be an environment variable in server.yml at all so all available instances for a given serviceId will
be returned and load balanced.

## Summary

The above solution replaces the ad hoc solution built today in the existing monolithic service. The
existing solution pass an environment header in the request to indicate the testing and there is a
piece code to determine which database to connect based on the header. There is no environment header
on production and there should be only one database connection as default on production. We cannot
follow this approach when we move to microservices architecture so we found a way to resolve it at 
the framework level instead. 

  
 
