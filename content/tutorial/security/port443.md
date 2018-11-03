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

Unless your service is running in a dedicated locked down VM, we don't recommend running as root - it should be run as its own user. For most of our customers, they have F5 facing the Internet and it can route the traffic to the IP and port combination. When deploying light-4j instance to the cloud, ene solution is to front light-4j with a server such as HAProxy or Nginx, and let it proxy requests to light-4j, but this requires maintaining the another installation and reduce the performance dramatically as well. In situations where you are wanting to run light-4j on port 443, but you do not want to setup a proxy server you can use iptables on Linux to forward traffic.


### Prerequisites

In order to forward traffic from 443 to 8443, first you must ensure that iptables has allowed traffic on all 2 of these ports. Use the following command to list the current iptables configuration:

```
iptables -L -n
```

You should should see in the output entries for 443 and 8443. 

If you don't see entries for these ports, then you need to run commands (as root or with sudo) to add those ports. For example, if you see none of these and need to add them all, you would need to issue the following commands:

```
sudo iptables -I INPUT 1 -p tcp --dport 8443 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT
```

Note that I used -I INPUT 1. In a lot of iptables documentation/examples, you will see -A INPUT. The difference is that -A appends to the list of rules, while -I INPUT 1 inserts before the first entry. Usually when adding new accept ports to iptables configuration, you want to put them at the beginning of the ruleset, not the end. Run iptables -L -n again and you should now see entries for these 2 ports.

### Forwarding

Once the before and after forwarding ports are allowed, you can now run the command to forward port 443 traffic to 8443. The commands look like this:

```
sudo iptables -A PREROUTING -t nat -p tcp --dport 443 -j REDIRECT --to-port 8443
```

Once these rules are set and confirmed with iptables -L -n, and once your light-4j instance is up and running on port 8443, attempt to access your instance on port 443 instead of 8443. It should work and your URL should stay on port 443 - in other words, it should not get redirected to 8443. The fact that forwarding from 443 to 8443 should remain hidden from the client.


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

See https://help.ubuntu.com/community/IptablesHowTo for other Ubuntu options. There are many other resources describing this; please consult your system's documentation or search on the internet for information specific to your flavor of Linux.

If you are unsure at all about what kind of system you have, consult that system's documentation on how to update iptables configuration.



