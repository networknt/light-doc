---
title: "Free Let's Encrypt Certificate"
date: 2018-10-30T05:12:53-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

If you are using light-router as a public facing access point on a static IP or expose your services to the public, it makes perfect sense to use a CA-signed certificate instead of self-signed. If you don't want to foot the cost of the certificate, Let's Encrypt provides free certificates which can be recognized by almost all browsers. 

The following is the process to get a certificate on my public facing VMs.

There are a lot of different ways to use Let's Encrypt, and the certbot is the most convenient one. 

### DNS

The first step is the ensure that your domain name is matching the VM you are using. You need to ensure that in your DNS setup, there should be an `A` record points to your IP with the right subdomain name. If you are using Cloudflare, you need to disable the pass through so that it points to your IP directly. 


### Firewall 

You need to enable the firewall to allow public access to your VM on port 80. 

```
sudo ufw allow 80/tcp
```

### Certificate

Go to the following site https://certbot.eff.org and choose software `None of the above` and System is Ubuntu 18.04 in my case. Follow the instruction on the site and the following is what I have done on my virtual machine. 

Install the certbot from PPA

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot
```

Since your server architecture doesn't yet support automatic installation you'll have to use the certonly command to obtain your certificate.

```
sudo certbot certonly
```
- Select Spin up a temporary web server
- Enter email address
- Agree the Terms of Service
- Select if you want to share your email
- Select your domain name


The cert and key will be written to your /etc/letsencrypt/live folder

```
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/faucet.taiji.io/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/faucet.taiji.io/privkey.pem
   Your cert will expire on 2019-01-28. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

Now you have the fullchain certificate and the private key. The next step is to create a server.keystore that can be used by light-4j service. 

If you want to create a certificate with multiple domains, it is easier to list the domains in the commmand line. 

```
sudo certbot certonly -d taiji.io -d faucet.taiji.io -d lightapi.net -d demo.taiji.io
```

If you want to create a new cert with other domain added, you can add the --duplicate option. 

```
sudo certbot --duplicate certonly -d taiji.io -d faucet.taiji.io -d lightapi.net -d demo.taiji.io

```

As on the website, we have port 443 occupied already, when getting the cert, we want to use port 80 instead of 443. 

```
sudo certbot --duplicate --preferred-challenges http certonly -d taiji.io -d faucet.taiji.io -d lightapi.net -d demo.taiji.io
```

If you have an existing certificate for only one domain, it will ask you if you want to expend it. Select (E) to proceed. 


