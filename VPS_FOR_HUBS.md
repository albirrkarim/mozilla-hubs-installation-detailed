# Introduction

This Article is about hosting the mozilla hubs into vps / self hosted server.

You must understand [installing mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) before you read this article.

## Warning, this tutorial is not complete yet.

<br>

# Requirement

Knowledge

Before you must understand the basic first. look this youtube video [Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

![Up skill](/docs_img/excercise.gif)

- Basic CLI linux
- Github Actions
- Remoting VPS via ssh

Software

- VPS with ubuntu based

Other

- Enough sleep
- Coffee
- Calm music
- Public IP

<br>

# Installation

## 1. Install Nginx

Login with SSH

```
ssh username@your_IP
```

Update all to the latest

```
sudo su
apt-get update
apt-get upgrade
```

[Install Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

## 2. Install Firewall and setting up

### 2.1 Install

We are using ufw. firewall is some software to blocking / allow port.

Please look at this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

### 2.2 Add Rules

Now we must allow some port

**I dont know exactly what is this port haha, just allow it**
```
ufw allow http,https,ssh,OpenSSH,'Nginx full'
```

**The hubs port**

![System Overview](/docs_img/System_Overview.jpeg)

**Allow for tcp and udp protocol**

```
ufw allow proto tcp from any to any port 4000,4443,8080,9090,8989
```

```
ufw allow proto udp from any to any port 4000,4443,8080,9090,8989
```

**The Dialog port**

```
ufw allow proto tcp from any to any port 40000:49999
```

```
ufw allow proto udp from any to any port 40000:49999
```

Now you can see all rules with

```
ufw status numbered
```

or delete with 
```
ufw delete <number>
```


## 3. Setting up https for your domain

Installing certbot and use letsencrypt.

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python3-certbot-nginx
```

Generating certificates

```
sudo certbot --nginx -d example.com -d www.example.com
```

It will generate certificate for that domain. so where the file?

```
sudo su
cd /etc/letsencrypt/live/example.com
```

In here you will see the cert.pem chain.pem fullchain.pem privkey.pem.

We will need that file so download it.

Goto `/etc/letsencrypt/live` we will zip the folder `example.com` and move it too the `/`

```
zip -r temp.zip example.com
cd ../../../
```

Then download it with `scp` command.

```
sudo scp username@your_ip:/temp.zip /Downloads
```

it will download from server and save to the Downloads folder. `/Downloads` is the destination where you save the file.

## 4. Install resources monitoring for vps

We are using [webmin](https://www.webmin.com/deb.html). this for monitoring server resources like CPU, RAM, and memory.
We dont know the resource usage hubs yet, so we must monitoring it.

Maybe on installation of web min you got some error like me

**Can't connet on port 10000**

the default port of webmin is 10000 but some ISP block that port. so we need to change with 1000. [see](https://serverfault.com/a/578397)

and don't forget about we must allow rules on `ufw` firewall

```
sudo ufw allow proto tcp from any to any port 1000
```

**HTTPS error**

If you can't open webmin ( port 1000) on chrome, please use mozilla firefox

Go to [this web](https://www.inmotionhosting.com/support/product-guides/cloud-server/ssl-errors-and-https-in-webmin/)

Upload your certificates file that we have download before.

## 5. Make sure it on private repository

The entire hubs (reticulum,dialog,hubs,spoke) make it private repo. just to be sure it safe.

## 7. Install Dependencies

**Install database**

[Install postgres on linux ubuntu](https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu)

**Elixir and Erlang (Elixir 1.12.3 and erlang version 23.3)**

You can installing those with follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Becareful about the version of elixir and erlang.

you can check the current elixir and erlang with 
```
asdf current
```

If you got problem when installing erlang you must install their deps.



**Install Process Management**

Later we will run all node js server like dialog, hubs, hubs admin, spoke on different port. so we need process management for running that on the background.

See [install pm2](https://pm2.keymetrics.io/)


## 8. Setting up github actions

I assume you 

### 8.1 Elixir based
On reticulum 



### 8.2 Node js based

Goto the action tab and new workflow -> choose node js

#### Hubs

Setting up yml file like this

#### Hubs admin

Setting up yml file like this

#### Spoke

Setting up yml file like this

#### Dialog

Setting up yml file like this

<br>
<br>
<br>

## Thank you for read this, I will update this soon. Give me star for supporting me.

## If you have a questions feel free to open and issue

<br>
<br>
<br>

# Useful reference

[Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)
