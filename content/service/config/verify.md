---
title: "Config Server API verify on local environment"
date: 2018-07-04T14:48:25-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---


### Build and start the service on local docker test environment


1. Build the config server module

```
cd ~/networknt/light-config-server
git pull origin develop
mvn clean install

```

2. Build Hybrid-command from light-portal

```
cd ~/networknt/light-portal
git pull origin develop
mvn clean install

```

3. Copy the Hybrid service required jar files for light-docker

```
cd ~/networknt/light-docker
git pull origin develop
cp ~/networknt/light-config-server/target/config-server-1.5.19.jar ~/networknt/light-docker/hybrid-services/service
cp ~/networknt/light-portal/hybrid-command/target/hybrid-command.jar ~/networknt/light-docker/hybrid-services/service

```

4. Start backend db  and hybrid service from light-docker

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-hybrid-service.yml up

```


### Config value security


-- Some sensitive values need to be encrypted

 When the light-config-server hybrid service started, system will create an encrpt/decrpt key file on the files system (by default will be in the user home folder).

 File name:  light-config-server.conf


System admin should trigger the Initial server service first after light-config-server service start to run to set the encrpt/decrpt key into the key file ( light-config-server.conf)


--Initial server by add secret key for the service (in real settig, it should be done by admin only):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"initialize-server","version":"0.1.0","data":{"key":"light-config"}}
```


### Verify the config server


1. Test create service:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"create-service","version":"0.1.0","data":{"serviceId":"config-service-1.1.0001","encryptionAlgorithm":"AES","templateRepository":"git@github.com:networknt/light-config-server.git","serviceOwner":"Google","version":"1.1.1","profile":"DEV/DIT","refreshed":false}}
'

```

The create service request return the generated config service with config service id, for example:  0000016543342c47-0242ac1300050000.

Use the generated config service Id for the following tests;



2. Test create config server value: key-value pair:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"create-service-value","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000","key":"server/serviceId","value":"1222222"}}
'
```


3. Test config server values: key-value pair array:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"create-service-values","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000","values":[{"key":"server/enableHttps","value":"true"}, {"key":"server/httpsPort","value":"true"}]}}
'
```

4. If the config value in to be encrpted in the database create the config server secret values:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: b9803960-e5b2-42c7-afc1-5d071bc57d96' \
  -d '{"host":"lightapi.net","service":"config","action":"create-service-secrets","version":"0.1.0","data":{"configServiceId":"00000165680f7e16-0242ac1200060000","values":[{"key":"server/buildNumber","value":"cibcBuild"}, {"key":"server/truststoreName","value":"keytab:112"}]}}
'
```

5. update service:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"update-service","version":"0.1.0","data":{"serviceId":"config-service-1.1.0001","encryptionAlgorithm":"AES","templateRepository":"https://github.com/chenyan71/light-config-template.git","serviceOwner":"networknt","version":"1.1.1","profile":"DEV/DIT","refreshed":false}}
'
```


6. update config value:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"update-service-value","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000","key":"server/serviceId","value":"fass-22222"}}
'
```

6. update config values (list of key-value pairs):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"update-service-values","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000","values":[{"key":"server/enableHttps","value":"false"}, {"key":"server/httpsPort","value":"false"}]}}
'
```


7. update config secret values (list of key-value pairs):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 278c94c0-59e6-43c6-852e-381b30f6fdf4' \
  -d '{"host":"lightapi.net","service":"config","action":"update-service-secrets","version":"0.1.0","data":{"configServiceId":"00000165680f7e16-0242ac1200060000","values":[{"key":"server/buildNumber","value":"cibc-gow-1.1.2"}, {"key":"server/truststoreName","value":"keycache:112"}]}}
'

```


8. query  service:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"query-service","version":"0.1.0","data":{"serviceId":"config-service-1.1.0001","profile":"DEV/DIT","version":"1.1.1"}}
'
```

10. query  config value by specified service id and key:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"query-service-value","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000","key":"server/truststoreName"}}
'
```


11. query  config values (config value key-pair list for specified service):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"query-service-values","version":"0.1.0","data":{"configServiceId":"00000165904d7f75-0242ac1200030000"}}
'
```



12. delete service:

```
POST /api/json HTTP/1.1
Host: localhost:8443
Content-Type: application/json
Cache-Control: no-cache
{"host":"lightapi.net","service":"config","action":"delete-service","version":"0.1.0","data":{"serviceId":"config-service-1.1,1","encryptionAlgorithm":"AES","templateRepository":"git@github.com:networknt/light-config-server.git","version":"1.1.1","profile":"DEV/DIT","refreshed":false}}

```


13.delete config value by specified service id and key:

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"delete-service-value","version":"0.1.0","data":{"configServiceId":"00000165680f7e16-0242ac1200060000","key":"server/buildNumber"}}
'
```


14.delete config values (delete ALL config values for specified service):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"delete-service-values","version":"0.1.0","data":{"configServiceId":"00000165680f7e16-0242ac1200060000"}}
'
```

15.retrieve config values (config.zip file):

```
curl -X POST \
  https://localhost:8443/api/json \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{"host":"lightapi.net","service":"config","action":"retrieve-config","version":"0.1.0","data":{"serviceId":"config-service-1.1.0001","profile":"DEV/DIT","version":"1.1.1"}}
'
```