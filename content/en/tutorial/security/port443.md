---
title: "Running ligh-4j service on port 443"
date: 2018-11-02T14:19:45-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

The default light-codegen generated project will have HTTPS and HTTP2 enabled on port 8443. Typically, HTTPS servers run on port 443. But this port is considered privileged on Unix/Linux systems, and the process using them must be owned by root. 

Unless your service is running in a dedicated locked down VM, we don't recommend running as root - it should be run as its own user. For most of our customers, they have F5 facing the Internet and it can route the traffic to the IP and port combination. When deploying light-4j instance to the cloud, one solution is to front light-4j with a server such as HAProxy or Nginx, and let it proxy request to light-4j, but this requires maintaining the another installation, which reduces the performance dramatically as well. In situations where you are wanting to run light-4j on port 443, but you do not want to setup a proxy server you can use iptables on Linux to forward traffic.


### Prerequisites

In order to forward traffic from 443 to 8443, first you must ensure that iptables has allowed traffic on all 2 of these ports. Use the following command to list the current iptables configuration:

```
sudo iptables -L -n
```

You should see in the output entries for 443 and 8443. 

If you don't see entries for these ports, then you need to run commands (as root or with sudo) to add those ports. For example, if you see none of these and need to add them all, you would need to issue the following commands:

```
sudo iptables -I INPUT 1 -p tcp --dport 8443 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT
```

Note that I used -I INPUT 1. In a lot of iptables documentation/examples, you will see -A INPUT. The difference is that -A appends to the list of rules, while -I INPUT 1 inserts before the first entry. Usually when adding new accept ports to iptables configuration, you want to put them at the beginning of the ruleset, not the end. Run iptables -L -n again and you should now see entries for these 2 ports.

If ufw is enabled on your VM, you can issue the following command instead of iptables. 

```
sudo ufw allow 443/tcp
sudo ufw allow 8443/tcp
```

### Forwarding

Once the before and after forwarding ports are allowed, you can now run the command to forward port 443 traffic to 8443. The commands look like this:

```
sudo iptables -t nat -I PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
```

Once these rules are set and confirmed with iptables -L -n, and once your light-4j instance is up and running on port 8443, attempt to access your instance on port 443 instead of 8443. It should work and your URL should stay on port 443 - in other words, it should not get redirected to 8443. The fact that forwarding from 443 to 8443 should remain hidden from the client.

### Check Configuration

After you change something, you might wondering what is the rules at the moment. You can export the current rules to a text file to check it out. And you can restore the rules with the text file. 

```
sudo iptables-save > iptables_rules.txt
```

You can use command line to check the rules. 

```
sudo iptables -L -n
```

As PREROUTING rule is part of the NAT, the above command line won't show the PREROUTING rule, to see them, use the following command. 

```
sudo iptables -L -n -t nat
```

### Remove Rules

If you make a mistake and add/insert an incorrect PREROUTING rule, you can delete it with the following commands. 

First you need to find out which line number the rule is. 

```
sudo iptables -t nat --line-numbers -L
```

Then delete the rule in PREROUTING by line number. 

```
sudo iptables -t nat -D PREROUTING 5
```

As the line number might be changed after removing one record, you need to rerun the --line-numbers command if you want to remove another record. 


### Save Configuration

Using the iptables command to change port configuration and routing rules only changes the current, in-memory configuration. It does not persist between restarts of the iptables service. So, you need to make sure you save the configuration to make the changes permanent.

Saving the configuration is slightly different between RedHat-based and Debian-based systems. On a RedHat-based system (Fedora, CentOS, RHEL, etc), issue the following command:

```
sudo iptables-save > /etc/sysconfig/iptables
```

On a Debian-based system (Debian, Ubuntu, Mint, etc), issue the following command:

```
sudo sh -c "iptables-save > /etc/iptables.rules"
```

The iptables-restore command will need to be executed manually, or your system configured to automatically run it on boot, against the /etc/iptables.rules file you have created, in order for your iptables configuration to be retained across reboots. On Ubuntu fastest way is to install iptables-persistent after configuring iptables - it will automatically create necessery files from current configuration and load them on boot.

```
sudo apt-get install iptables-persistent
```

If the package is installed already. 

```

sudo dpkg-reconfigure iptables-persistent
```

See https://help.ubuntu.com/community/IptablesHowTo for other Ubuntu options. There are many other resources describing this; please consult your system's documentation or search on the internet for information specific to your flavor of Linux.

If you are unsure at all about what kind of system you have, consult that system's documentation on how to update iptables configuration.

### Troubleshooting ufw

When you have ufw enabled on your system, sometimes, above command won't work. You need to disable the ufw and enable it again.  

First, remove all the rules.

```
sudo ufw disable
sudo ufw enable
sudo iptables -t nat -I PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
sudo ufw allow 8443/tcp
sudo ufw allow 443/tcp
```

If you are unsure how the rules are working or what is blocking your request, you can monitor the log of iptables. 

```
sudo tail -f /var/log/syslog
```

### Local access

The above setup works on a remote server that is accessed from the Internet. However, if you want to access the 443 port on the same host expected to be forwarded to the 8443, you need to add the following command line. In most cases, it is your Linux desktop for development. 

```
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -o lo -j REDIRECT --to-port 8443
```

The rule adds a similar rule to the OUTPUT table that redirects packets outgoing to port 443 on the loopback interface (-o lo).

We don't save this rule, so it will be removed after the computer is restarted. If you want to remove the rule without restarting the computer for local testing, follow the steps below.

Find the line number of the rule with the following command. 

```
sudo iptables -t nat --line-numbers -L
```

You will have something like below for Chain OUTPUT. 

```
Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DOCKER     all  --  anywhere            !localhost/8          ADDRTYPE match dst-type LOCAL
2    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 8443
```

You OUTPUT REDIRECT might have different line numbers other than 2. The following command can remove the rule given the line number is 2 on my computer. 

```
sudo iptables -t nat -D OUTPUT 2

```

You can confirm the rule is removed by issue the query again. The above OUTPUT REDIRECT should be removed already. 

```
sudo iptables -t nat --line-numbers -L
```

** Warning ** Please beware that the iptables rule can impact the docker container to access the Internet on your host. So please don't make this rule persistent on your local. 

For more information, please visit this [issue](https://stackoverflow.com/questions/34319369/cannot-connect-to-https-443-from-a-docker-image)

This rule will block your docker to access any HTTPS sites on the Internet. So if you encounter any problem with your docker build or docker run, you need to check your iptables rule.

