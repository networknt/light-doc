---
title: "Getting started light-eventuate-4j"
date: 2017-12-07T20:14:25-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This tutorial will show you how to start light-eventuate-4j services with docker-compose. It is
very convenient with developers who want to do integration test on his/her laptop or desktop. For
official test environment and production, it is not recommended to use docker-compose or docker. 
These services should be installed in data center or virtual machines to form clusters. 

### Prepare Environment

Kafka needs a network IP address for ADVERTISED_HOST_NAME and it cannot be localhost or 127.0.0.1
as localhost within docker container means the docker container itself. It must be an address that
is accessible within docker container. It should be the hostname of your computer or the real network
IP address of your computer that runs docker. 

There is a script in light-docker that can try to find the hostname and export it as DOCKER_HOST_IP
but it is not reliable as hostname might just mapped to 127.0.0.1 on some computers. The best way to
do that would be manually find out the IP address on your computer and export it.

##### Manually export DOCKER_HOST_IP

On Mac/Linux, issue the following command

```
ifconfig
```

You will see a list of network interfaces in the output. Depending on your network setup, the interface
name might be en0 on Mac and enp4s0 on Linux. You basically need to find the IP start with 192.XXX.XXX.XXX
or 10.XXX.XXX.XXX and copy it into the clipboard.

The next step is to export it to an environment variable DOCKER_HOST_IP.

```
export DOCKER_HOST_IP=198.168.1.120
```

The above approach is the most reliable way to get the right IP address and on my Mac Book Pro it is the
only working way as the computer name is not mapped to a real IP address somehow.

If you want to setup a static IP for your Mac, then you can follow the steps below to set it up. Docker 
for Mac has [networking limitations](https://docs.docker.com/docker-for-mac/networking/) and you need to 
follow the steps below to set it up.

```
sudo ifconfig lo0 alias 10.200.10.1/24  # (where 10.200.10.1 is some unused IP address)
export DOCKER_HOST_IP=10.200.10.1
```

##### Export DOCKER_HOST_IP with script

This work on my Linux desktop as the hostname joy is mapped to 198.168.1.120 in /etc/hosts. The address
of Mac Book Pro is changing all the time depending on the WIFI network it connected to.

Here is the IPV4 section in /etc/hosts on my desktop.

```
127.0.0.1	localhost
192.168.1.120   joy
```

To run the script, we need to checkout the light-docker repository from networknt.

Assume that we have a working directory called networknt under user home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
source set-docker-host-ip.sh
```

After that, you can double check the DOCKER_HOST_IP from the following command line on the same
terminal. 

```
echo $DOCKER_HOST_IP
```

In order to confirm that the IP address or hostname is valid, you can ping it and see if you have
responses. 

##### Put the export into .bashrc

If you are using a desktop which has ba static IP address, then you can put the export statement
into .bashrc under your home directory. This can eliminate the manual step and ensure that you can
run the docker-compose from any terminal. 

Here is a section of my .bashrc file on my desktop.

```
export PATH=$PATH:/home/steve/tool/apache-maven-3.5.0/bin
export M2_HOME=/home/steve/tool/apache-maven-3.5.0
export M2=%M2_HOME%\bin
export DOCKER_HOST_IP=192.168.1.120
```

Note that above manual export and script will only work on the terminal window with the export and
you have to start the docker-compose with the exact terminal in order to work as the environment
variable only associates to that particular terminal instance. 

Right after you updated .bashrc, you can run the following command and then run the docker-compose
from the same terminal. The next time you restart your computer, you can run the docker-compose in
any terminal.

```
cd ~
source .bashrc
```

This approach only works with desktop as it has a static IP. If you are using a laptop, most likely
you will have to use the first option to export the IP manually. 


### Start eventuate services

If you have used script to export DOCKER_HOST_IP, then you have clone the light-docker to your
workspace already. If not, let's do it here. 

```
cd ~/networknt
git clone https://github.com/networknt/light-docker.git
cd light-docker
```

Now let's start the docker-compose-eventuate.yml from light-docker folder. Before we run the compose,
we need to export the DOCKER_HOST_IP on this terminal unless you have updated .bashrc file.

After double check the DOCKER_HOST_IP environment variable, you can issue the following command to
start Kafka, Zookeeper and Mysql all together.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-eventuate.yml up 
```

It will take several minutes to get all three services to start. Once all started, you can start the
CDC server. 

### Start CDC server for eventuate

Eventuate CDC is a server that monitors the events table in the database and send the events to Kafka for other
services to subscribe. To start eventuate-cdc-server, you can use a docker-compose file in light-docker.

```
cd ~/networknt/light-docker
docker-compose -f docker-compose-cdcserver-for-eventuate.yml up
```

Another way to start CDC server is to start from the source code build. In this case, we need to compile
light-eventuate-4j repo. Unless you are doing update for the CDC server, you should use the above 
docker-compose to start CDC server for eventuate. 

```
cd ~/networknt
git clone https://github.com/networknt/light-eventuate-4j.git
cd light-eventuate-4j
mvn clean install -DskipTests
cd eventuate-cdc-server
java -jar target/eventuate-cdc-server.jar
```

Now we have the entire infrastructure up and running, we can start some of the demo applications. 
