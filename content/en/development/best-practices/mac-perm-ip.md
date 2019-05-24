---
title: "Mac Permanent IP"
date: 2018-04-07T23:22:45-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

We have a team of developers using MacBook Pro for development, and they have to travel from one office to another. When the laptop joins the WIFI network at a different location, a brand new IP address is allocated. This should be an issue for most docker containers running on the laptop except Kafka as it has to advertise to DOCKER_HOST_IP. 

We can use ifconfig to find the real IP address and export it with the following command. 

```
export DOCKER_HOST_IP=10.200.10.1
```

After that, we can stop and restart the docker-compose for eventuate local environment. It is doable but not very convenient. To allow developers to start the eventuate docker-compose only once and use it until the laptop is rebooted, we need to find a way to set up a fixed IP address for the MacBook Pro. 

On Mac, you can use the following command to create an alias network interface and assign a fixed IP to it. 

```
sudo ifconfig lo0 alias 10.200.10.1/24  # an unused IP address
```

Once this is done, you can use the above export command to bind this IP to DOCKER_HOST_IP and pass it to the Kafka docker container. It works until you restart your laptop. How can we make it into the startup script so that we don't need to remember that every time you restart your computer?

We can put the export command into .profile file under home directory. But the command to create an alias is executed with sudo. 


For this special case, we need to utilize launchd to kick off a short script with the ifconfig command you used above.

Here is a sample .plist file, saved as com.user.lo0-loopback.plist (can be saved anywhere as it will be copied to the appropriate directory later).

```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE plist PUBLIC -//Apple Computer//DTD PLIST 1.0//EN http://www.apple.com/DTDs/PropertyList-1.0.dtd > 
<plist version="1.0"> 
<dict> 
  <key>Label</key> 
  <string>com.user.lo0-loopback</string> 
  <key>ProgramArguments</key> 
  <array> 
    <string>/sbin/ifconfig</string> 
    <string>lo0</string> 
    <string>alias</string> 
    <string>10.200.10.1</string> 
  </array> 
  <key>RunAtLoad</key> <true/> 
  <key>Nice</key> 
  <integer>10</integer> 
  <key>KeepAlive</key> 
  <false/> 
  <key>AbandonProcessGroup</key> 
  <true/> 
  <key>StandardErrorPath</key> 
  <string>/var/log/loopback-alias.log</string> 
  <key>StandardOutPath</key> 
  <string>/var/log/loopback-alias.log</string> 
</dict> 
</plist>
```

Next, move it to the /Library/LaunchDaemons/ directory so it's kicked off at boot (will be run as root) and set the correct permissions.

```
$ sudo cp com.user.lo0-loopback.plist /Library/LaunchDaemons/ 
$ sudo chmod 0644 /Library/LaunchDaemons/com.user.lo0-loopback.plist 
$ sudo chown root:wheel /Library/LaunchDaemons/com.user.lo0-loopback.plist
```

Then load it with launchctl

```
$ launchctl load /Library/LaunchDaemons/com.user.lo0-loopback.plist
```
Reboot and your lo0 loopback should have an alias IP assigned to it that will be persistent across reboots.


With above steps done, you can put the export command into the .profile. Here is the .profile file for my MacBook Pro. 

```
#export JAVA_HOME=`/usr/libexec/java_home -v 9.0.4`
export JAVA_HOME=`/usr/libexec/java_home -v 1.8.0_101`
export PATH="$HOME/.cargo/bin:$PATH"
export DOCKER_HOST_IP=10.200.10.1
```

Now you can reboot your laptop and double check if the network interface alias is still there with the following command

```
ifconfig
```

The result should be something like this. 

```
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
    options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
    inet 127.0.0.1 netmask 0xff000000 
    inet6 ::1 prefixlen 128 
    inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
    inet 10.200.10.1 netmask 0xffffff00 
    nd6 options=201<PERFORMNUD,DAD>
```

After confirming this alias is working all the time, you can refer to [light-eventuate-4j getting started] to start the docker-compose for eventuate local development environment. 

[light-eventuate-4j getting started]: /tutorial/eventuate/getting-started/

