---
title: "Cluster Install"
date: 2018-08-01T10:01:36-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

### Introduction

HashiCorp Consul provides distributed, highly available, datacenter-aware service discovery and configuration for microservices. It allows services to register itself and client to discover interested services. 

In the following tutorial, we are going to install a three nodes Consul server. It is recomended that you have 3 or 5 Consul servers running in each datacenter and each instance should be installed in a physical machine or a VM. 

Here is the list of machines with Consul server to be installed. 

```
198.55.49.188 Consul server / initial bootstrap server
198.55.49.187 Consul server
198.55.49.186 Consul server
```

### Install Bootstrap Server

Download and extract consul binary and place it into /usr/local/bin so that it is accessible from the PATH. Here is the [download page][] for all Operating Systems. We are going to install the Linux 64-bit version.

```
wget https://releases.hashicorp.com/consul/1.2.2/consul_1.2.2_linux_amd64.zip
sudo unzip ./consul_1.2.2_linux_amd64.zip -d /usr/local/bin
```

Create the user and following directories. 

```
sudo adduser consul
sudo mkdir -p /etc/consul.d/{bootstrap,server,client}
sudo mkdir /var/consul
sudo chown consul:consul /var/consul
```

### Creating the Bootstrap Configuration 

We want to implement some encryption to the whisper protocol that consul uses. It has this functionality built in using a shared secret system. 

```
consul keygen
T9hm99zu12SPcZcutuQN4w==
```

### Create Bootstrap File

sudo vi /etc/consul.d/bootstrap/config.json

```
{
"bootstrap": true,
"server": true,
"datacenter": "dc1",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true
}
```

### Create Server Config

sudo cp /etc/consul.d/bootstrap/config.json /etc/consul.d/server

update the config for the server. Note that the bootstrap is false and start_join is added. 

sudo vi /etc/consul.d/server/config.json

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.188",
"bind_addr": "198.55.49.188",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.188" },
"ports": {"http": 8500 },
"start_join": ["198.55.49.187", "198.55.49.186"]
}
```

### Create Server Config for other nodes

The encrypt key must be the same for all of the participants in the cluster, so copying the file has already taken care of that requirement for us. Keep this in mind when creating new configurations. Copy the contents of this configuration file to the other machines that will be acting as your consul servers. Place them in a file at /etc/consul.d/server/config.json just as you did in the first host.

The only value you need to modify on the other hosts is the IP addresses that it should attempt to connect to. You should make sure that it attempts to connect to the first server instead of its own IP. For instance, the second server in our example would have a file that looks like this:


sudo vi /etc/consul.d/server/config.json


```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.187",
"bind_addr": "198.55.49.187",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.187" },
"ports": {"http": 8500 },
"start_join": ["198.55.49.188", "198.55.49.186"]
}
```

The third node will have server config like this. 

sudo vi /etc/consul.d/server/config.json

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.186",
"bind_addr": "198.55.49.186",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.186" },
"ports": {"http": 8500 },
"start_join": ["198.55.49.188", "198.55.49.187"]
}

```


### Create Start Script

On all nodes, create start script. As we are using public IP address, we need to bind it in the command line. Please remember to change the bind ip in each node. 

sudo vi /etc/systemd/system/consul.service

```
[Unit]
Description=consul agent
Requires=network-online.target
After=network-online.target
[Service]
EnvironmentFile=-/etc/sysconfig/consul
Environment=GOMAXPROCS=2
Restart=on-failure
ExecStart=/usr/local/bin/consul agent -config-dir=/etc/consul.d/server -rejoin -ui -data-dir=/var/consul
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target
```

Then, run the following commands.

```
sudo systemctl daemon-reload
sudo systemctl start consul
sudo systemctl enable consul
```


### Start Cluster

On a server that contains the bootstrap configuration file (the first node), use su to change to the consul user briefly. We can then call consul and pass in the bootstrap directory as an argument:

```
su consul
consul agent -config-dir /etc/consul.d/bootstrap -bind 198.55.49.188
```

Please note that we have -bind in the command line as we don't have private IP address but only public IP. 

During the bootstrap, this server will be elected as the leader. 

### Start Other Nodes

On your other consul servers, start the server. 

```
sudo systemctl start consul
```


These servers will connect to the bootstrapped server, completing the cluster. At this point, we have a cluster of three servers, two of which are operating normally, and one of which is in bootstrap mode, meaning that it can make executive decisions without consulting the other servers. This is not what we want. We want each of the servers on equal footing. Now that the cluster is created, we can shutdown the bootstrapped consul instance and then re-enter the cluster as a normal server.

To do this, hit CTRL-C in the bootstrapped server's terminal: 

Now, exit back into your root session and start the consul service like you did with the rest of the servers:

```
sudo systemctl start consul
```

Now, we can see how many members in the cluster. 

consul members

```
Node           Address             Status  Type    Build  Protocol  DC   Segment
198-55-49-186  198.55.49.186:8301  alive   server  1.2.2  2         dc1  <all>
198-55-49-187  198.55.49.187:8301  alive   server  1.2.2  2         dc1  <all>
198-55-49-188  198.55.49.188:8301  alive   server  1.2.2  2         dc1  <all>
```

To enable these nodes to access each other, the firewall rules might need to be defined. The following is on my server.

```
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
8500/tcp                   ALLOW       Anywhere                  
8300/tcp                   ALLOW       Anywhere                  
8301/tcp                   ALLOW       Anywhere                  
8500/tcp (v6)              ALLOW       Anywhere (v6)             
8300/tcp (v6)              ALLOW       Anywhere (v6)             
8301/tcp (v6)              ALLOW       Anywhere (v6)
```

### Enable ACLs on the Consul Servers

The first step for bootstrapping ACLs is to enable ACLs on the Consul servers in the ACL datacenter. In this example, we are configuring the following:

1. An ACL datacenter of "dc1", which is where these servers are
2. An ACL master token of "bio324k3lje23"; see below for an alternative using the /v1/acl/bootstrap API
3. A default policy of "deny" which means we are in whitelist mode
4. A down policy of "extend-cache" which means that we will ignore token TTLs during an outage

Here's the corresponding JSON configuration file. Please add the content into the /etc/consul.d/server/config.json

```
{
"acl_datacenter": "dc1",
"acl_master_token": "bio324k3lje23",
"acl_default_policy": "deny",
"acl_down_policy": "extend-cache"
}
```

The servers will need to be restarted to load the new configuration. Please take care to start the servers one at a time, and ensure each server has joined and is operating correctly before starting another

To restart the server.

```
sudo systemctl restart consul
```

### Generate an Agent Token

We can create a token using the ACL API, and the ACL master token we set in the previous step:

```
curl \
--request PUT \
--header "X-Consul-Token: bio324k3lje23" \
--data \
'{
"Name": "Agent Token",
"Type": "client",
"Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"write\" }"
}' http://198.55.49.188:8500/v1/acl/create
```

The result would be something like the following.

```
{"ID":"3f5f1cef-2966-4964-73c5-7ebeb21ba337"}
```

The above curl command is assuming that http is used. Once the TLS is enabled on the Consul server, you need to use the following command line instead. 

```
curl -k --request PUT --header "X-Consul-Token: bio324k3lje23" --data '{"Name": "Agent Token","Type": "client","Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"write\" }"}' https://198.55.49.188:8500/v1/acl/create
```

### Create an Agent Token

In Consul 0.9.1 and later you can also introduce the agent token using an API, so it doesn't need to be set in the configuration file:

```
curl \
--request PUT \
--header "X-Consul-Token: bio324k3lje23" \
--data \
'{
"Token": "3f5f1cef-2966-4964-73c5-7ebeb21ba337"
}' http://198.55.49.188:8500/v1/agent/token/acl_agent_token
```

With that ACL agent token set, the servers will be able to sync themselves with the catalog.

In most of our case, we are going to generate a token instead creating a token with known id. 

### ACL Policy

In light platform, there are two different clients that need to communicate with Consul and the policies are different. 

* Service

For a service, it needs to register itself to the consul during startup and deregister itself during shutdown. Also, it need to call health check endpoint to declare that it is healthy. Some of the services might need to call other services so that it needs to lookup other services as a client. 

Except the lookup query, service needs to have a token that policy defined as write. 

For example, we are using the following command to create a token that is writable for service. As write policy has default read covered, this token can read the service catalog as well. 

```
curl -k --request PUT --header "X-Consul-Token: bio324k3lje23" --data '{"Name": "Agent Token","Type": "client","Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"write\" }"}' https://198.55.49.188:8500/v1/acl/create

```

* Original Client

For an original client, there is no need to update anything on the Consul server, it only need a read-only policy for service. Here is an example to create a read-only token for a client. 

```
curl -k --request PUT --header "X-Consul-Token: bio324k3lje23" --data '{"Name": "Agent Token","Type": "client","Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"read\" }"}' https://198.55.49.188:8500/v1/acl/create

```

For an organzation with a lot of services and clients, it is recommended that we create a service token and client token for one LOB instead of create separate tokens for each individual service. 


### Enable TLS

Above installation is suitable for non-production environment to test initial Consul connectivity issues. This setup and configuration should be suitable for a production environment, however TLS/SSL must be implemented on the WebUI and communication ports to future ensure secure communication. 

https://www.consul.io/docs/agent/encryption.html#configuring-tls-on-an-existing-cluster

Self Signed SSL with Consul

https://www.digitalocean.com/community/tutorials/how-to-secure-consul-with-tls-encryption-on-ubuntu-14-04

http://russellsimpkins.blogspot.ca/2015/10/consul-adding-tls-using-self-signed.html


After the configuration is done, here is the consul1 config.json

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.188",
"bind_addr": "198.55.49.188",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.188" },
"ports": {"http": -1, "https": 8500 },
"ca_file": "/etc/consul.d/ssl/ca.cert",
"cert_file": "/etc/consul.d/ssl/consul.cert",
"key_file": "/etc/consul.d/ssl/consul.key",
"verify_incoming": false,
"verify_outgoing": true,
"acl_datacenter": "dc1",
"acl_master_token": "bio324k3lje23",
"acl_default_policy": "deny",
"acl_down_policy": "extend-cache",
"start_join": ["198.55.49.187", "198.55.49.186"]
}

```

Please note that verify_incoming is false in this server so that browser can connect without providing certificate. 


Here is the consul2 config.json

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.187",
"bind_addr": "198.55.49.187",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.187" },
"ports": {"http": -1, "https": 8500 },
"ca_file": "/etc/consul.d/ssl/ca.cert",
"cert_file": "/etc/consul.d/ssl/consul.cert",
"key_file": "/etc/consul.d/ssl/consul.key",
"verify_incoming": true,
"verify_outgoing": true,
"acl_datacenter": "dc1",
"acl_master_token": "bio324k3lje23",
"acl_default_policy": "deny",
"acl_down_policy": "extend-cache",
"start_join": ["198.55.49.188", "198.55.49.186"]
}

```

Here is the consul3 config.json

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"198.55.49.186",
"bind_addr": "198.55.49.186",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "198.55.49.186" },
"ports": {"http": -1, "https": 8500 },
"ca_file": "/etc/consul.d/ssl/ca.cert",
"cert_file": "/etc/consul.d/ssl/consul.cert",
"key_file": "/etc/consul.d/ssl/consul.key",
"verify_incoming": true,
"verify_outgoing": true,
"acl_datacenter": "dc1",
"acl_master_token": "bio324k3lje23",
"acl_default_policy": "deny",
"acl_down_policy": "extend-cache",
"start_join": ["198.55.49.188", "198.55.49.187"]
}

```

Once you switch your browser to https from http, the existing ACL token is not working anymore and you have to input the token from ACL tab again. 


As we are using public IP for consul servers, we have to protect other nodes to join the same cluster. To do that we have setup the firewall to only allow the connection from another two nodes. Here is the firewall status on the first consul server.

```
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
8500/tcp                   ALLOW       Anywhere                  
Anywhere                   ALLOW       198.55.49.187             
Anywhere                   ALLOW       198.55.49.186             
22/tcp                     ALLOW       Anywhere                  
8500/tcp (v6)              ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6) 
```

### Install with private IP addresses. 

In the case that you are using private IP addresses, you don't need to specify the exact ip to bind. For example, you can use the following config.json. 

```
{
"bootstrap": false,
"server": true,
"datacenter": "dc1",
"client_addr":"0.0.0.0",
"data_dir": "/var/consul",
"encrypt": "T9hm99zu12SPcZcutuQN4w==",
"log_level": "INFO",
"enable_syslog": true,
"addresses": { "http": "0.0.0.0" },
"ports": {"http": -1, "https": 8500 },
"ca_file": "/etc/consul.d/ssl/ca.cert",
"cert_file": "/etc/consul.d/ssl/consul.cert",
"key_file": "/etc/consul.d/ssl/consul.key",
"verify_incoming": false,
"verify_outgoing": true,
"acl_datacenter": "dc1",
"acl_master_token": "bio324k3lje23",
"acl_default_policy": "deny",
"acl_down_policy": "extend-cache",
"start_join": ["198.55.49.187", "198.55.49.186"]
}
```

### Useful Commands

The following are some useful commands that access consul APIs.

```
curl -k -H "X-Consul-Token: 3f5f1cef-2966-4964-73c5-7ebeb21ba337" https://198.55.49.188:8500/v1/agent/checks


curl -k -H "X-Consul-Token: 3f5f1cef-2966-4964-73c5-7ebeb21ba337" https://198.55.49.188:8500/v1/catalog/services


curl -k -H "X-Consul-Token: 3f5f1cef-2966-4964-73c5-7ebeb21ba337" https://198.55.49.188:8500/v1/catalog/service/com.networknt.apid-1.0.0


curl -k -H "X-Consul-Token: 3f5f1cef-2966-4964-73c5-7ebeb21ba337" https://198.55.49.188:8500/v1/health/checks/com.networknt.apid-1.0.0

```


[download page]: https://www.consul.io/downloads.html
