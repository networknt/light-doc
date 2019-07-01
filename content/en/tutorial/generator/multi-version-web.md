---
title: "Multiple Version Codegen-Web"
date: 2019-05-27T12:30:48-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

As you know, we have developed a service in light-codegen called codegen-web to generate light-4j projects on the cloud without build and run light-codegen locally. As we have multiple release branches, we need to support users to generate code for the release they are working. This requires that we must support multiple APIs at the same time. 

Codegen-web is built on top of light-hybrid-4j, which is very easy to support multiple versions of the same service running at the same time. However, it requires the UI to be implemented much more complicated to call different versions. To make the UI much simpler, we are going to run two or more codegen-web servers. At this moment, we are going to support release 1.6.x and 2.0.x at the same time. 

To walk through how the codegen-web is built, we are going to split this tutorial into three different stages.

### Develop and Test

This is for the light-codegen developers to work on the functionality of the codegen web and the view of codegen-web. There is no need to start multiple instances of the backend servers. There is no light-router and consul server evolved. 

To work on the codegen-web, you need to first checkout and build the light-codegen. Let's assume you are working on the master branch by default. 

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
cd light-codegen
mvn clean install
```

Once the build is done, we can start the codegen-web server.

```
cd codegen-web
java -jar target/codegen-web-server-shaded.jar
```

This will start the codegen-web server that is listening to 8080 port on HTTP. All the configuration files are using the default from resources/config.

Once the server is up and running, let's start the React UI view.

```
cd ~/networknt/light-codegen/codegen-web/view
yarn install
npm start
```

This will start the nodejs server with React hot reloading. All the calls with prefix /codegen will be proxied to the backend server on 8080. The nodejs server is listening to port 3000. 

Now you can update the UI code or update the backend code and test them together. 

### Local Compose

Although we have two instances of codegen web, we are sharing some code so that we can avoid changing code in two different branches. First, the UI code is the same for both 1.6.x and 2.0.x. Second both instances are using the same codegen-server from light-portal. It is a docker container that can host both Java 8 1.6.x codegen-web jars and Java 11 2.0.x codegen-web jars. 

To start the docker-compose locally with two instances of codegen-web, light-router and the UI served by the light-router, we have created a folder in light-config-test repo. You can checkout the repo from networknt organization. 

```
cd ~/networknt
git clone https://github.com/networknt/light-config-test.git
cd light-config-test/light-codegen
ls -l
```

The following files or directories in the folder.

```
steve@freedom:~/networknt/light-config-test/light-codegen$ ls -l
total 28
drwxr-xr-x 5 steve steve 4096 Jun 30 13:55 1_6_x
drwxr-xr-x 5 steve steve 4096 Jun 30 13:55 2_0_x
drwxrwxr-x 3 steve steve 4096 Jun 25 16:12 codegen-web
-rw-rw-r-- 1 steve steve  764 Jun 30 13:56 docker-compose-local.yaml
-rw-rw-r-- 1 steve steve  767 Jun 28 17:45 docker-compose.yaml
-rw-rw-r-- 1 steve steve 2180 Jun 30 13:54 README.md
drwxr-xr-x 4 steve steve 4096 Jun 30 13:55 router

```

The `docker-compose-local.yaml` is used to start the local docker-compose. 

```
version: "2"

services:
  2_0_x:
    image: networknt/codegen-server-1.0.0
    volumes:
      - ./2_0_x/local:/config
      - ./2_0_x/service:/service
    ports:
      - 8440:8440
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    

  1_6_x:
    image: networknt/codegen-server-1.0.0
    volumes:
      - ./1_6_x/local:/config
      - ./1_6_x/service:/service
    ports:
      - 8439:8439
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    

  light-router:
    image:  networknt/light-router:latest
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
    ports:
    - 443:8443
    volumes:
    - ./router/local:/config
    - ./codegen-web/build:/codegen-web/build

```

Let's walk through this file to explain all the details. You will notice that service 2_0_x and 1_6_x are sharing the same image which is the codegen-server from light-portal. For service 2_0_x, the config is read from ./2_0_x/local folder and all the Java 11 codegen jar files are loading from the ./2_0_x/service folder. As the jar files are not checked into the GitHub, here is a list of jar files in the service folder.

```
steve@devops:~/networknt/light-config-test/light-codegen/2_0_x/service$ ls -l
total 1400
-rw-r--r-- 1 steve steve   8033 Jun 30 17:56 codegen-core.jar
-rw-r--r-- 1 steve steve   3707 Jun 30 17:56 codegen-fwk.jar
-rw-r--r-- 1 steve steve  22401 Jun 30 17:56 codegen-web.jar
-rw-r--r-- 1 steve steve 468823 Jun 30 17:56 light-eventuate-4j-generator.jar
-rw-r--r-- 1 steve steve 177956 Jun 30 17:56 light-graphql-4j-generator.jar
-rw-r--r-- 1 steve steve 187058 Jun 30 17:56 light-hybrid-4j-generator.jar
-rw-r--r-- 1 steve steve 554372 Jun 30 17:56 light-rest-4j-generator.jar
```

These files are copied from light-codegen modules after the build from master branch with JDK 11.

For service 1_6_x, the config is read from ./1_6_x/local folder and all the Java 8 codegen jar files are loaded from the ./1_6_x/service folder. The name of the jars are the same as above; however, these jars are build from 1_6_x branch with JDK 8. 

One environment variable DOCKER_HOST_IP is passed in as we are using host network and this environment variable will be used by the light-4j server to register to consul with the right IP address. 

For me, I have the following line in my .profile in my user's home directory. 

```
export DOCKER_HOST_IP=192.168.1.144
```

For the light-router instance, the config is loaded from ./router/local and the UI React production code is loaded from ./codegen-web/build. To create the build folder, you need to go to the light-codegen/codegen-web/view and run the following build command. 

```
cd ~/networknt/light-codegen/codegen-web/view
npm run build
```

There would be a build folder created under view. Copy the build folder to the `~/networknt/light-config-test/light-codegen/codegen-web`

As we are using consul to register the codegen-web and discovery these two services from light-router, we need to start the consul server locally first. You need to checkout light-docker first if you haven't done so. 

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
docker-compose -f docker-compose-consul.yml up -d
```

As our web server is running as a virtual host, we need to access it throught the domain `codegen.lightapi.net` from the browser. For local testing, we can update the /etc/hosts to add an entry like this.

```
192.168.1.144   codegen.lightapi.net
```

Please remember to comment out this line when we move to the cloud in the next step. Otherwise, the access is always point to the local server, not the cloud server. 

Once above steps are done, start the docker-compose. 

```
cd ~/networknt/light-config-test/light-codegen
docker-compose -f docker-compose-local.yaml
```

If everything starts correctly, you can access the UI from the browser.

```
https://codegen.lightapi.net/
```


### Cloud Compose

For the official production deployment, the light-codegen is deployed to the `devops` server with a docker-compose. Here is the content of the docker-compose.yaml

```
version: "2"

services:
  2_0_x:
    image: networknt/codegen-server-1.0.0
    volumes:
      - ./2_0_x/config:/config
      - ./2_0_x/service:/service
    ports:
      - 8440:8440
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    

  1_6_x:
    image: networknt/codegen-server-1.0.0
    volumes:
      - ./1_6_x/config:/config
      - ./1_6_x/service:/service
    ports:
      - 8439:8439
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    

  light-router:
    image:  networknt/light-router:latest
    environment:
      - STATUS_HOST_IP=${DOCKER_HOST_IP}
    network_mode: host    
    ports:
    - 443:8443
    volumes:
    - ./router/config:/config
    - ./codegen-web/build:/codegen-web/build

```

It is very similar to the local compose. The difference is that the config is loaded from config folder instead of local folder. Here is the steps to get it deployed to the devops server.

```
ssh devops
```

Once logged in, add the DOCKER_HOST_IP to the .profile

```
vi .profile
# export DOCKER_HOST_IP=192.168.1.144
source .profile
```

Please change the IP to your IP on the server.

The next step is to enable ufw to allow the consul cluster to access this server on 8440 and 8439 on health endpoint. 

```
sudo ufw status
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow from xxx.xxx.xxx.186
sudo ufw allow from xxx.xxx.xxx.187
sudo ufw allow from xxx.xxx.xxx.188
```

The above firewall rules allow the ssh from anywhere and allow the three nodes from the consul cluster to access this server on all ports. Please note that you need to allow the ssh once the ufw is enabled, otherwise, you cannot login to this server with ssh anymore once you are disconnected. 

We need to copy the jdk11 and jdk8 jar files to the service folder under 2_0_x and 1_6_x with scp. 

For example, the following comamnd is for 2_0_x from the local


```
cd ~/networknt/light-config-test/light-codegen/2_0_x/service
scp *.jar steve@devops:/home/steve/networknt/light-config-test/light-codegen/2_0_x/service

```

As the router is listening to port 8443, we need to setup the iptables to forward 443 to 8443 on this server. Please follow this [tutorial](/tutorial/security/port443/) to set it up. 

Once everything is done, let's start the docker-compose

```
docker-compose up -d
```

To learn how to use the codegen-web, please follow the video below.

{{< youtube amBapgXCExg >}}