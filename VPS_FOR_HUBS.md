# Introduction

This article is about hosting the Mozilla hubs into VPS / self-hosted server. 

Hubs provide easy deploy on AWS. its just work. But I don't want to buy some server on AWS. because i have my own server. so how to do that?

I spend 4 days trying to install hubs on VPS.

Basically im not a dev ops person. im react js and laravel little bit fullstack.

**You must try** [**installing Mozilla hubs on local**](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) **before you read this article. If not you will get lost**.

For the entire hubs (reticulum, dialog, hubs, spoke) make it private repo. just to be sure it is safe.

Star this repo for supporting me. and if you are interested in web 3D like this you can follow my [github account](https://github.com/albirrkarim).

I try to make software overview, architecture, and tables on the database. you can see my [figma project](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Paypal](https://paypal.me/AlbirrKarim)

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

# Warning

Sometimes there's a step that I'm not writing down on my tutorial. you know some little hack that we sometimes do but we forget about it.

You can give it a try. and give me messages if there is a missing step/bug that I forget to write it down

# Requirement

**Knowledge**

Before you must understand the basics first. look at this youtube video [Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

![Up skill](/docs_img/excercise.gif)

- Basic CLI Linux
- Github Actions
- Remoting VPS via ssh
- little about protocol TCP and UDP

**Software**

- VPS with ubuntu based. I suggest using the LTS version

**Other**

We will go on a long journey, so this is an important requirement

- Enough sleep
- Fruit juice, a few coffees
- Calm music

<br>

# Installing

Table of content

1. [Install Dependencies](#1-install-dependencies)
2. [Install Firewall and Setting Up](#2-install-firewall-and-setting-up)
3. [Setting up HTTPS for Your Domain](#3-setting-up-https-for-your-domain)
4. [Setting up Github Actions](#4-setting-up-github-actions)
5. [Set Your Public IP and Domain](#5-set-your-public-ip-and-domain)
6. [Run all](#6-run-all)
7. [Setting up NGINX](#7-setting-up-nginx)

Optional

- [Resources Monitoring for VPS (optional)](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/RESOURCE_MONITORING.md)

## 1. Install Dependencies

Login with SSH. if the ssh is taking long or forever to connect, try using VPN. it will work.

```
ssh username@your_IP
```

**Update all to the latest update**

```
sudo su
apt-get update
apt-get upgrade
```

**We using Nginx for server**

[Install Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

**Install database**

[Install Postgres on Linux ubuntu](https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu)

**Elixir and Erlang (Elixir 1.12.3 and erlang version 23.3)**

You can be installing those with `asdf` please follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Be careful about the version of elixir and erlang, you must exact the same version with this tutorial.

you can check the current elixir and erlang with

```
asdf current
```

If you got a problem when installing erlang you must install their dependencies.

**Install Process Management**

Later we will run all node js servers like dialog, hubs, hubs admin, spoke on a different port. so we need process management for running that in the background.

See [install pm2](https://pm2.keymetrics.io/)

## 2. Install Firewall and set up

### 2.1 Install

We are using ufw. a firewall is some software to block/allow ports.

Please look at this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

### 2.2 Add Rules

Now we must allow some port.

**I don't know exactly what is this port haha, just allow it**

```
ufw allow http,https,ssh,OpenSSH,'Nginx full'
```

**The hubs port**

![Up skill](/docs_img/port.png)

**Allow for TCP and UDP protocol**

For now, I don't really understand about ports let's just allow it

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

**Install certbot**

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python3-certbot-nginx
```

**Generating certificates**

```
sudo certbot --nginx -d example.com -d www.example.com
```

It will generate the certificate for that domain. so where is the file?

```
sudo su
cd /etc/letsencrypt/live/example.com
```

In here you will see the `cert.pem` `chain.pem` `fullchain.pem` `privkey.pem`

**Backup the certificates**

We will need that file to give SSL certificate to the [monitoring resource](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/RESOURCE_MONITORING.md)

Goto `/etc/letsencrypt/live` we will zip the folder `example.com` and move it to the `/`

```
zip -r temp.zip example.com
cd ../../../
```

Then download it with `scp` command.

```
sudo scp username@your_ip:/temp.zip /Downloads
```

It will download from the server and save to the Downloads folder. `/Downloads` is the destination where you save the file.

**Make it accessible**

[Change the permision](https://www.google.com/search?q=give+user+folder+permissions+linux) for `live` folder so it can be accessed from hubs projects.

Inside `/etc/letsencrypt/live/` i remove the `example.com` folder and make a new
`example.com` folder which contains all cert files that we have a backup before. because I don't know why my files are empty.

**Automatically renew certificates**

Follow [this](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx)

## 4. Setting up Github Actions

Make sure you watch [this](https://www.youtube.com/watch?v=X3F3El_yvFg) first

### 4.1 Elixir based

#### Reticulum

On your reticulum repository go to the actions tab on Github. and create a new workflow then choose elixir.

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
 
      - name: Install dependencies
        run: mix deps.get

      - name: Compile production release
        run: MIX_ENV=prod mix release

      - name: Start server
        run: |
          MIX_ENV=prod mix compile
          PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

### 4.2 Node js based

One workflow (.yml) is for one repo

Do this step to each repository:

Goto the action tab -> new workflow -> choose node js.

It will create a default .yml file. Then replace the content with this:

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
      - name : Stop server
        run: | 
          pm2 stop hubs_server
          pm2 stop hubs_admin_server

      - name: Install hubs deps
        run: npm i --force

      - name: Install hubs admin deps
        run: |
          cd admin/
          pwd
          npm i --force
          ls
      
      - name: Start hubs server
        run: pm2 start hubs_server

      - name: Start hubs admin server
        run: pm2 start hubs_admin_server
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

      - name: Stop server
        run: pm2 stop spoke_server

      - name: Install deps
        run: yarn install

      - name: Start server
        run: pm2 start spoke_server
```

#### Dialog

Only dialog im not using github actions. because i got some problem setting up github action for this.

### 4.3 Add self-hosted

Make sure you watch [this](https://www.youtube.com/watch?v=X3F3El_yvFg) first

Above you can see `runs-on: self-hosted` it means the command below it, will run on your server.


#### Folder for the action runner

I suggest you to preparing a clean folder structure on your VPS

Before you add the actions runner. Make a new empty folder like this. just run 

```
mkdir folder_name
```

Remember! this is an empty folder, not your "cloned repo"

```
home
    your_username
        hubs-actions-runner     <- wrap up with some folder
            hubs                <- where you put gihub action runner
            reticulum           <- where you put gihub action runner
            dialog              <- where you put gihub action runner
            spoke               <- where you put gihub action runner
```

Okay, take a look at this picture below
 
![Setting](/docs_img/action_runner.png)

There you can follow the tutorial provided by GitHub

Skip the create folder step. because we have [already make empty folder](#folder-for-the-action-runner) for that.

the get in on each folder

for example reticulum

`cd /hubs-actions-runner/reticulum`

then follow the tutorial provided by GitHub (see the images above) like download the tar file -> config -> run 

the tutorial from GitHub action runner is running with `sudo ./run.sh` it will run. but if we close the terminal it will die.

On the runner folder on your server. you can see `svh.sh` alongside with `run.sh`

run

```
sudo ./svh.sh install
```

then

```
sudo ./svh.sh start
```

For your information. GitHub action runner will automatically pull from GitHub to your server in the folder:

```
/hubs-actions-runner/hubs/_work/hubs/hubs/IN_HERE
```

```
/hubs-actions-runner/reticulum/_work/reticulum/reticulum/IN_HERE
```

etc


### 4.4 Setting path yarn, mix, elixir

[see](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/SETTING_PATH_GITHUB_RUNNER.md)



## 5. Set Your Public IP and Domain

Attention! For this section, you will need to change `example.com` with your domain. don`t just copy and paste it.

### 5.1 Reticulum

Let me explain how I do that. I copy the `config/dev.exs` and name it with `prod.exs` then I modify it a little.

Take a look at [prod.exs](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/source_code/prod.exs)

<br>

**The part you should pay attention to**

**- The host**

Fill it with your domain

**- Endpoint config**

```elixir
config :ret, RetWeb.Endpoint,
```

The [path](#3-setting-up-https-for-your-domain) of keyfile and certfile

Change the `secret_key_base` with your key which result from `mix phx.gen.secret`

**- Database config**

```elixir
# Configure your database
config :ret, Ret.Repo,
config :ret, Ret.SessionLockRepo,
```

DB name is `ret_dev`

the host is `localhost`

**- Janus load status**

```elixir
config :ret, Ret.JanusLoadStatus,
```

the port is 4443

**- Storage**

```
config :ret, Ret.Storage,
  host: "https://#{host}:4000",
  storage_path: "/home/admin/hubs-actions-runner/reticulum/storage",
  ttl: 60 * 60 * 24
```

By default, the configuration for storage is `storage/`. it means like this

```
/home/your_username/hubs-actions-runner/reticulum/_work/reticulum/reticulum/storage
```

If we set the storage path to the inside repo action runner like above it will auto remove by git repository synchronization.

so we need make new folder on `/home/admin/hubs-actions-runner/reticulum`

```
mkdir -p storage/dev
```

To show the current path in the terminal you can use `pwd` command

### 5.2 Dialog

On `package.json` make new command `prod`

Change the IP with your public IP and the domain of course

```
MEDIASOUP_LISTEN_IP=123.xxx.xxx.xxx MEDIASOUP_ANNOUNCED_IP=123.xxx.xxx.xxx HTTPS_CERT_FULLCHAIN=/etc/letsencrypt/live/example.com/fullchain.pem HTTPS_CERT_PRIVKEY=/etc/letsencrypt/live/example.com/privkey.pem DOMAIN=example.com node index.js
```

### 5.3 Hubs

In `package.json` make a new script named `prod`

```
webpack-dev-server --mode=production --env.prodVps --https --cert /etc/letsencrypt/live/example.com/cert.pem --key /etc/letsencrypt/live/example.com/privkey.pem
```

In `webpack.config.js` add this

```js
if (argv.mode === "production") {
  if (env.prodVps) {
    // Production on VPS
    const your_domain = "example.com";
    
    // We dont use the reticulum port 4000 because later we will proxy pass from port 443 to 4000

    Object.assign(process.env, {
      HOST_IP: your_domain,
      SHORTLINK_DOMAIN: `${your_domain}`,
      HOST: your_domain,
      RETICULUM_SOCKET_SERVER: your_domain,
      CORS_PROXY_SERVER: `${your_domain}`,
      NON_CORS_PROXY_DOMAINS: `${your_domain},dev.reticulum.io`,
      BASE_ASSETS_PATH: `https://${your_domain}:8080/`,
      RETICULUM_SERVER: your_domain,
      POSTGREST_SERVER: "",
      ITA_SERVER: "",
      UPLOADS_HOST: `https://${your_domain}`,
    });
  }
}
```

### 5.4 Hubs admin

on `package.json` add new command named `prod`

```bash
webpack-dev-server --mode=production --env.prod=true --https --cert /etc/letsencrypt/live/example.com/cert.pem --key /etc/letsencrypt/live/example.com/privkey.pem
```

In `webpack.config.js` add this

```js
if (env.prod) {
  const your_domain = "example.com";
  Object.assign(process.env, {
    HOST: your_domain,
    RETICULUM_SOCKET_SERVER: your_domain,
    CORS_PROXY_SERVER: "hubs-proxy.com",
    NON_CORS_PROXY_DOMAINS: `${your_domain},dev.reticulum.io`,
    BASE_ASSETS_PATH: `https://${your_domain}:8989/`,
    RETICULUM_SERVER: your_domain,
    POSTGREST_SERVER: "",
    ITA_SERVER: "",
    HOST_IP: your_domain,
  });
}
```

### 5.5 Spoke

on `package.json` add new command named `prod`

```
cross-env NODE_ENV=production HOST_IP=example.com BASE_ASSETS_PATH=https://example.com:9090/ webpack-dev-server --mode production --https --cert /etc/letsencrypt/live/example.com/cert.pem --key /etc/letsencrypt/live/example.com/privkey.pem
```

Edit the `.env.prod`

```
HUBS_SERVER="example.com:4000"
RETICULUM_SERVER="example.com"
FARSPARK_SERVER="farspark.reticulum.io"
CORS_PROXY=""
NON_CORS_PROXY_DOMAINS="example.com:4000,reticulum.io"
ROUTER_BASE_PATH="/spoke"
GITHUB_ORG="mozilla"
GITHUB_REPO="spoke"
GITHUB_PUBLIC_TOKEN="ghp_SAFEPB2zzes9TEpAOSx2McNjJLQ1GXLBES2FsfWU"
IS_MOZ="false"
CORS_PROXY_SERVER=""
```

## 6. Run all

### 6.1 Elixir based

**Reticulum**

Basically, we can start manually with this. But [previously](#reticulum) we have done set auto-deploy

To start manually you can use this command, this will start the reticulum server in the background.

```bash
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

For checking the reticulum is running use this command to list the process which runs on port 4000

```
lsof -n -i4TCP:4000
```

To stop the process you can kill with PID

```
kill -9 PID
```

Or with [single command](https://stackoverflow.com/a/55115797)

```
(lsof -ti:4000) && kill -9 $(lsof -ti:4000)
```

### 6.2 Node js based

#### 6.2.1 Process manager

If we run the node js project we use terminal. if we close that terminal the node js server will die. so we need to run that server in the background. with `pm2` we can manage the process like start, stop, restart, watch server logs.

Useful pm2 command:

**Start a process on background**

```
pm2 start EXECUTABLE --name PROCESS_NAME -- SOME_PARAMS
```

**Watching server logs**

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

The `PROCESS_NAME` params can be changed to `all` to affect all process

#### 6.2.2 Run node js server

**Hubs, Hubs admin, Dialog**

Move to `hubs` repo files location

```
cd /hubs-actions-runner/hubs/_work/hubs/hubs
```

and try to run first with

```
npm run prod
```

if its ok (no error), then using pm2

with

```
pm2 start npm --name hubs_server -- run prod
```

do that also to the dialog, hubs admin

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

#### 6.2.3 Make sure it runs well

Make sure the process name is same as in [.yml files](#node-js-based)

```
dialog_server
spoke_server
hubs_server
hubs_admin_server
```

run 

```
pm2 status
```

![status](/docs_img/pm2_status.png)


### 6.3 Auto start your all server on startup

Basically, all processes will be killed if your server is rebooted.

thanks to [this](https://stackoverflow.com/questions/45412600/pm2-process-disappears-after-reboot), with pm2 run:

```
pm2 startup
```

![status](/docs_img/pm2_startup.png)

then run 

```
pm2 save
```

For the reticulum, we need to make a bash script for automatic start

Thanks to [this](https://stackoverflow.com/questions/12973777/how-to-run-a-shell-script-at-startup)

On root dir, make .sh file named `start_reticulum_server.sh`

```bash
#!/bin/bash

echo "STARTING RETICULUM SERVER"
export PATH=$PATH:/home/admin/.asdf/shims
echo $PATH

cd /hubs-actions-runner/reticulum/_work/reticulum/reticulum
(lsof -ti:4000) && kill -9 $(lsof -ti:4000)
MIX_ENV=prod mix release --overwrite
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server

sleep 3

(lsof -ti:4000) && echo "Server started"
```

Info: the export path is for crontab knows the `mix` command location


Then make the .sh file is executable 

```
chmod +x start_reticulum_server.sh
```

Open the crontab with this command

```
crontab -e
```

and paste this command on the bottom then quit and save

```
# For starting reticulum server
@reboot /home/admin/start_reticulum_server.sh >> /home/admin/start_reticulum.log 2>&1
```

from the command above it means on reboot crontab will run command `start_reticulum_server.sh` and save the logs to `start_reticulum.log`

## 7. Setting up NGINX

We must pass everything to port 4000

So setting up
`proxy_pass` on Nginx

Open the Nginx config file with

```
sudo nano /etc/nginx/sites-available/default
```

And replace the content with this code

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name example.com;

    location / {
        # match everything
        rewrite ^\/(.*)$ /$1 break;
        # Proxy passing to port 4000
        proxy_pass https://example.com:4000;

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
        #match everything
        rewrite ^\/(.*)$ /$1 break;
        # Proxy passing to port 4000
        proxy_pass https://example.com:4000;

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

## IF you have questions feel free to open an issue

<br>
<br>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

<br>
<br>

## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Basic Information for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

<br>
<br>

# Useful reference

[Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

[TCP vs UDP](https://www.youtube.com/watch?v=uwoD5YsGACg)
