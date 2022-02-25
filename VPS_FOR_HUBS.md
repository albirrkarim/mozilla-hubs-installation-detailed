# Introduction

This article is about hosting the mozilla hubs into vps / self hosted server. I spend 4 days of trying to install hubs on vps.

You must understand [installing mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) before you read this article.

For the entire hubs (reticulum,dialog,hubs,spoke) make it private repo. just to be sure it safe.

## Warning, this tutorial is not complete yet. so dont follow this tutorial

<br>

# Requirement

**Knowledge**

Before you must understand the basic first. look this youtube video [Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

![Up skill](/docs_img/excercise.gif)

- Basic CLI linux
- Github Actions
- Remoting VPS via ssh
- little about protocol TCP and UDP

**Software**

- VPS with ubuntu based. I suggest using LTS version

**Other**

We will go on a long journey, so this is important requirement

- Enough sleep
- Fruit juice, few coffee
- Calm music

<br>

# Installing

Table of content

1. [Install Nginx](#1-install-nginx)
2. [Install Firewall and Setting Up](#2-install-firewall-and-setting-up)
3. [Setting up HTTPS for Your Domain](#3-setting-up-https-for-your-domain)
4. [Install Dependencies](#4-install-dependencies)
5. [Setting up Github Actions](#5-setting-up-github-actions)
6. [Run all](#6-run-all)
7. [Setting up NGINX](#7-setting-up-nginx)
8. [Resources Monitoring for VPS (optional)](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/RESOURCE_MONITORING.md)


## 1. Install Nginx

Login with SSH. if the ssh is takes long or forever to connect, try using VPN. it will works.

```
ssh username@your_IP
```

Update all to the latest update

```
sudo su
apt-get update
apt-get upgrade
```

We using Nginx for server, and proxy pass

[Install Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

## 2. Install Firewall and setting up

### 2.1 Install

We are using ufw. firewall is some software to blocking / allow port.

Please look at this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

### 2.2 Add Rules

Now we must allow some port.

**I dont know exactly what is this port haha, just allow it**

```
ufw allow http,https,ssh,OpenSSH,'Nginx full'
```

**The hubs port**

![Up skill](/docs_img/port.png)

**Allow for tcp and udp protocol**

For now I don't really understand about ports lets just allow it

```
ufw allow proto tcp from any to any port 4443,8080,9090,8989
```

```
ufw allow proto udp from any to any port 4443,8080,9090,8989
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

## 3. Setting up HTTPS for Your Domain

Install certbot

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

In here you will see the `cert.pem` `chain.pem` `fullchain.pem` `privkey.pem`

We will need that file so download it.

Goto `/etc/letsencrypt/live` we will zip the folder `example.com` and move it to the `/`

```
zip -r temp.zip example.com
cd ../../../
```

Then download it with `scp` command.

```
sudo scp username@your_ip:/temp.zip /Downloads
```

it will download from server and save to the Downloads folder. `/Downloads` is the destination where you save the file.

## 4. Install Dependencies

**Install database**

[Install postgres on linux ubuntu](https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu)

**Elixir and Erlang (Elixir 1.12.3 and erlang version 23.3)**

You can installing those with `asdf` please follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Becareful about the version of elixir and erlang, you must exact same version with this tutorial.

you can check the current elixir and erlang with

```
asdf current
```

If you got problem when installing erlang you must install their deps.

**Install Process Management**

Later we will run all node js server like dialog, hubs, hubs admin, spoke on different port. so we need process management for running that on the background.

See [install pm2](https://pm2.keymetrics.io/)

## 5. Setting up Github Actions

### 5.1 Elixir based

#### Reticulum

On your reticulum repository goto actions tab on github. and create new workflow then choose elixir.

it will create `.github/workflow/elixir.yml`

change the content `elixir.yml` with

```yml
name: Elixir CI

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    name: Build and test
    runs-on: self-hosted

    env:
      ImageOS: ubuntu20

    steps:
      - uses: actions/checkout@v2
      - name: Set up Elixir
        uses: erlef/setup-beam@988e02bfe678367a02564f65ca2e37726dc0268f
        with:
          # Define the elixir version [required]
          elixir-version: "1.12.3"
          # Define the OTP version [required]
          otp-version: "23.1"
      - name: Restore dependencies cache
        uses: actions/cache@v2
        with:
          path: deps
          key: ${{ runner.os }}-mix-${{ hashFiles('**/mix.lock') }}
          restore-keys: ${{ runner.os }}-mix-
      - name: Install dependencies
        run: mix deps.get
      - name: Compile production release
        run: MIX_ENV=prod mix release
      - name: Start server
        run: |
          MIX_ENV=prod mix compile
          PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

### 5.2 Node js based

Goto the action tab and new workflow -> choose node js

#### Hubs

Setting up yml file like this

```yml
name: Node.js CI

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [16.x]

    steps:
      - uses: actions/checkout@v2
      - run: npm i --force
      - run: |
          cd admin/
          pwd
          npm i --force
          ls
      - run: pm2 restart hubs_server
      - run: pm2 restart hubs_admin_server
```

#### Spoke

Setting up yml file like this

```yml
name: Node.js CI

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [16.x]

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - run: yarn install
      - run: pm2 restart spoke_server
```

#### Dialog

Setting up yml file like this

```yml
name: Node.js CI

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [16.x]

    steps:
      - uses: actions/checkout@v2
      - run: npm i
```

### 5.3 Add self hosted

Above you can see `runs-on: self-hosted` it means the command bellow it, will run on your server.

![Setting](/docs_img/self-hosted.png)

There you can follow the tutorial provided by github

I suggest you to setting clean folder structure on your vps

```
root
    hubs_actions_runner     <- wrap up with some folder
        hubs                <- where you put gihub action runner
        reticulum           <- where you put gihub action runner
        dialog              <- where you put gihub action runner
        spoke               <- where you put gihub action runner
```

## 6. Run all

### 6.1 Elixir based

**Reticulum**

Basicaly we can start manually with this. But [previously](#reticulum) we have done set auto deploy

Start with this command

```bash
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

List the process which run on port 4000

```
lsof -n -i4TCP:4000
```

Kill with PID

```
kill -9 PID
```

Or with [single command](https://stackoverflow.com/a/55115797)

```
$(lsof -ti:4000) && kill -9 $(lsof -ti:4000)
```

### 6.2 Node js based

#### 6.2.1 Process manager

If we run node js project we using terminal. if we close that terminal the node js server will die. so we need run that server in background. with `pm2` we can manage process like start, stop, restart, watch server logs.

Useful pm2 command:

**Start a process on background**

```
pm2 start EXECUTABLE --name PROCESS_NAME -- SOME_PARAMS
```

**Wachting server logs**

```
pm2 logs
```

**See all process**

```
pm2 status
```

**Stoping process**

```
pm2 stop PROCESS_NAME
```

**Restart process**

```
pm2 restart PROCESS_NAME
```

The `PROCESS_NAME` params can be change to `all` to affect all process

#### 6.2.2 Run node js server

**Hubs, Hubs admin, Dialog**

Move to hubs action runner directory

and try to run first with

```
npm run prod
```

if its ok (no error), then using pm2

with

```
pm2 start npm --name hubs_server -- run prod
```

do that too to dialog, hubs admin

**Spoke**

Move to spoke action runner directory

Try to start the server

```
yarn prod
```

If no error then start with pm2

```
pm2 start yarn --name spoke_server -- prod
```

## 7. Setting up NGINX

We must pass everything to the port 4000

So setting up
`proxy_pass` on nginx

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name meta.dinus.ac.id;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        # try_files $uri $uri/ =404;

        #match everything
        rewrite ^\/(.*)$ /$1 break;
        # Proxy passing to port 4000
        proxy_pass https://meta.dinus.ac.id:4000;

        # Proxy Header
        #proxy_set_header        Host $host;
        #proxy_set_header        X-Real-IP $remote_addr;
        #proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_set_header        X-Forwarded-Proto $scheme;


        # The Important Websocket Bits!
        #proxy_http_version 1.1;
        #proxy_set_header Upgrade $http_upgrade;
        #proxy_set_header Connection "upgrade";

        #Give larger upstream buffers
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
    }
}


server {
    server_name example.com;

     location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;

        #match everything
        rewrite ^\/(.*)$ /$1 break;
        # Proxy passing to port 4000
        proxy_pass https://example.com:4000;

        # Proxy Header
        #proxy_set_header        Host $host;
        #proxy_set_header        X-Real-IP $remote_addr;
        #proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_set_header        X-Forwarded-Proto $scheme;


        # The Important Websocket Bits!
        #proxy_http_version 1.1;
        #proxy_set_header Upgrade $http_upgrade;
        #proxy_set_header Connection "upgrade";

        #Give larger upstream buffers
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
    }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

Restart NGINX

```bash
sudo systemctl restart nginx
```

<br>
<br>
<br>

## Thank you for read this, I will update this soon. Give me star for supporting me.
<br>

<br>
<br>

## IF you have a questions feel free to open an issue

<br>
<br>

# Useful reference

[Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)
[TCP vs UDP](https://www.youtube.com/watch?v=uwoD5YsGACg)
