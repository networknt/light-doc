---
title: "Debug Log Best Practices"
date: 2018-03-31T05:11:36-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

When you have an application that is not working as expected, the first thing would be checking the application logs to see if there are any errors there. Even if there are no errors, the information in logs can significantly help developers or supporters to understand how the application is working and where it stops. 

In production or any official testing environment, logging level must be set on error so that the application can focus on the business logic other than handling the logging. When an error occurs, it would be better to turn on the debug to collect more information about the suspicious subsystem or microservice.  

All light-4j libraries use slf4j and logback for logging and a logback.xml configuration template is provided for further customization. The default file is located in src/main/resources folder within the application. To overwrite the default file, you can make a copy to externalized config folder with different debugging level. 

The following is the command line to start the application in Dockerfile. As you can see, there is an option to specify logback.configurationFile in the command line. 

```
FROM openjdk:8-jre-alpine
ADD /target/petstore-1.0.1.jar server.jar
CMD ["/bin/sh","-c","java -Dlight-4j-config-dir=/config -Dlogback.configurationFile=/config/logback.xml -jar /server.jar"]
```

The above command tells the application to look for logback.xml in the /config folder first before settling down with the default file in src/main/resources folder. This setup gives us an opportunity to overwrite the default configuration with externalized logback.xml file.

When a dockerized application starts, a filesystem directory can be mapped to /config inside the container. Here is an example of a docker-compose file. 

docker-compose-cloud.yml

```
version: "2"
#
# Services
#
services:

  hybrid-command:
      build: ./hybrid-command/
      volumes:
        - ./hybrid-command/cloud/config:/config
        - ./hybrid-command/service:/service
      ports:
        - 8443:8443
      networks:
          - localnet

  hybrid-query:
      build: ./hybrid-query/
      volumes:
        - ./hybrid-query/cloud/config:/config
        - ./hybrid-query/service:/service
      ports:
        - 8442:8442
      networks:
          - localnet


# Networks
#
networks:
  localnet:
    # driver: bridge
    external: true
```

In the above compose file, you can put your logback.xml into ./hybrid-command/cloud/config folder to overwrite the default one. Once a new file is copied over, or an existing file is updated, you can restart the particular service with the following command. 

```
docker-compose -f docker-compose-cloud.yml restart hybrid-command
```

Now you can use docker-compose logs command to monitor on the console or get into the container to check the log file in /target folder. If [centralized logging][] is used, then you can search the logs on ElasticSearch with Kibana. 

You can also monitor the logs on the console/terminal. For example,  if you want to check the logs for cdcserver-eventuate page by page, you can use the following command line. 


```
docker-compose -f docker-compose-api.yml logs cdcserver-eventuate | more
```

[centralized logging]: /service/logging/