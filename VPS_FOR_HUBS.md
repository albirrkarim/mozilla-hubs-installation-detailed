# Introduction

This article is about hosting the Mozilla hubs into VPS / self-hosted server.

Hubs provide easy deploy on AWS. its just work. But I don't want to buy some server on AWS. because i have my own server. so how to do that?

I spend 4 days trying to install hubs on VPS.

Basically im not a dev ops person. im react js and laravel little bit fullstack.

Star this repo for supporting me. and if you are interested in web 3D like this you can follow my [github account](https://github.com/albirrkarim).

I try to make software overview, architecture, and tables on the database. you can see my [figma project](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

# Attention

You must try [installing Mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) before you read this article. If not you will get lost.

Why i told you for give try to running hubs project locally first (localhost)?

For me as a programmer i will make sure the code is work, features is work and the configuration also, in the local.

Because it will be more difficult to debug on the server rather than on local.

And you will understand how this project is going on.

**You can't instantly just clone and host hubs on the server.**

Please don't send me a message (about problems installing hubs on a server) on discord if you are not trying to run this locally first and succeed to do it.

# Advice

For the entire hubs (reticulum, dialog, hubs, spoke) make it private repo. just to be sure it is safe.

# Warning

Sometimes there's a step that I'm not writing down on my tutorial. you know some little hack that we sometimes do but we forget about it.

# Got Problem ?

Before you go far debugging days by days (like me).

You can instantly solve that, please read this:

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

# Requirement

**Knowledge**

Before you must understand the basics first. look at this youtube video [Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

![Up skill](/docs_img/excercise.gif)

- Basic CLI Linux
- Github Actions
- Remoting VPS via ssh
- VPN (if you remoting server on other country)
- little about protocol TCP and UDP

**Server**

- VPS with ubuntu based. Please use Ubuntu 20.04 LTS.
- at least 4GB of RAM and 50 GB Storage.

**Other**

We will go on a long journey, so this is an important requirement

- Enough sleep
- Fruit juice, a few coffees
- Calm music

<br>

# Installing

Table of content

1. [Install Dependencies](#1-install-dependencies)
2. [Install Firewall and Setting Up](#2-install-firewall-and-set-up)
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

### Update All

```
sudo su
apt-get update
apt-get upgrade
```

### Nginx

[Install Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

### Postgres database

[Install Postgres on Linux ubuntu](https://phoenixnap.com/kb/how-to-install-postgresql-on-ubuntu)

### Elixir and Erlang (Elixir 1.12.3 and erlang version 23.3)

You can be installing those with `asdf` please follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Be careful about the version of elixir and erlang, you must exact the same version with this tutorial.

you can check the current elixir and erlang with

```
asdf current
```

If you got a problem when installing erlang you must install their dependencies.

### Process Management

Later we will run all node js servers like dialog, hubs, hubs admin, spoke on a different port. so we need process management for running that in the background.

See [install pm2](https://pm2.keymetrics.io/)

### Mediasoup Dependency

Mediasoup is Cutting Edge WebRTC Video Conferencing. it used in dialog server. for RTC audio and video.

Look at [the mediasoup v3 Installation](https://mediasoup.org/documentation/v3/mediasoup/installation/#all-platforms)

you must install python 3 and make the default command `python` is calling `python3` see [this](https://dev.to/meetsohail/change-the-python3-default-version-in-ubuntu-1ekb)

## 2. Install Firewall and set up

### 2.1 Install

We are using ufw. a firewall is some software to block/allow ports.

Please look at this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

### 2.2 Add Rules

Now we must allow some port.

**I don't know exactly what is this port haha, just allow it**

```
ufw allow 'OpenSSH','Nginx full'
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

**Make it accessible**

```
sudo chmod 755 /etc/letsencrypt/live/ -R
sudo chmod 755 /etc/letsencrypt/archive/ -R

sudo chown $(whoami) /etc/letsencrypt/live/ -R
sudo chown $(whoami) /etc/letsencrypt/archive/ -R
```

then try to access it (THIS IS IMPORTANT)

```
cat /etc/letsencrypt/live/example.com/fullchain.pem
```

**Check Your Certificate Expiration**

letsencrypt give 90 days for certificate lifetime. so you must renew it every 3 month

```
certbot certificates
```

**Automatically renew certificates**

Make file `/home/admin/renew_cert.sh` and paste this.

```
#!/bin/sh

yes '2' | sudo certbot --nginx certonly -d example.com
sudo chown admin /etc/letsencrypt/live/example.com/privkey.pem
sudo chown admin /etc/letsencrypt/live/example.com/cert.pem

(lsof -ti:4000) && kill -9 $(lsof -ti:4000)
cd /home/admin/hubs-actions-runner/reticulum/_work/reticulum/reticulum
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
pm2 restart all
```

open cronjob with sudo. don't forget open cronjob with sudo because certbot renew requiring root access

```
sudo crontab -e
```

Paste this

```
# for renewing SSL certificate
0 12 * * * /home/admin/renew_cert.sh
```

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

      - name: Stop server
        run: pm2 stop hubs_admin_server

      - name: Install hubs deps
        run: npm i --force

      - name: Build for Hubs
        run: npm run build

      - name: Install hubs admin deps
        run: |
          cd admin/
          pwd
          npm i --force
          ls

      - name: Start server
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

      - name: Install deps
        run: yarn install

      - name: Build
        run: yarn build
```

#### Dialog

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
        run: pm2 stop dialog_server

      - name: Check version
        run: |
          node -v
          npm -v

      - name: Install dependency
        run: npm i

      - name: Start server
        run: pm2 start dialog_server

      - name: Check status
        run: pm2 status
```

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

On the runner folder on your server. you can see `svc.sh` alongside with `run.sh`

run

```
sudo ./svc.sh install
```

then

```
sudo ./svc.sh start
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
MEDIASOUP_LISTEN_IP=0.0.0.0 MEDIASOUP_ANNOUNCED_IP=123.xxx.xxx.xxx HTTPS_CERT_FULLCHAIN=/etc/letsencrypt/live/example.com/fullchain.pem HTTPS_CERT_PRIVKEY=/etc/letsencrypt/live/example.com/privkey.pem DOMAIN=example.com node index.js
```

### 5.3 Hubs

Open the `.default.env` and change / add the config

```
RETICULUM_SERVER="https://example.com"
BASE_ASSETS_PATH="https://example.com:8080/"
```

### 5.4 Hubs admin

Open the `.default.env` and change / add the config

```
# PostgREST server configured to allow administrative access to the db.
POSTGREST_SERVER="https://example.com/api/postgrest"
BASE_ASSETS_PATH="https://example.com:8989/"
```

<!-- on `package.json` add new command named `prod`

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
``` -->

### 5.5 Spoke

<!-- on `package.json` add new command named `prod`

```
cross-env NODE_ENV=production HOST_IP=example.com BASE_ASSETS_PATH=https://example.com:9090/ webpack-dev-server --mode production --https --cert /etc/letsencrypt/live/example.com/cert.pem --key /etc/letsencrypt/live/example.com/privkey.pem
``` -->

Edit this value the `.env.prod`

```
HUBS_SERVER="example.com"
RETICULUM_SERVER="example.com"
FARSPARK_SERVER="farspark.reticulum.io"
CORS_PROXY=""
NON_CORS_PROXY_DOMAINS="example.com:4000,reticulum.io"
ROUTER_BASE_PATH="/spoke"
IS_MOZ="false"
CORS_PROXY_SERVER=""
BASE_ASSETS_PATH="https://example.com:9090/"
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

**Dialog**

Move to `dialog` repo files location

```
cd /hubs-actions-runner/hubs/_work/dialog/dialog
```

and try to run first with

```
npm run prod
```

if its ok (no error), then using pm2

with

```
pm2 start npm --name dialog_server -- run prod
```

<!-- **Hubs Admin**

Hubs admin need web socket. run with `webpack-dev-server` so the server is live (not just static asset).

test it first with

```
npm run prod
```

make sure it work

then

```
pm2 start npm --name hubs_admin_server -- run prod
``` -->

#### 6.2.3 Serve with nginx for static assets

For better memory usage we don't need serve this static asset with `webpack-dev-server`

**Hubs, Hubs Admin, Spoke**

For webpack based, you can compile the production asset with this command:

```
npm run build
```

then it will resulting static asset like .html, .js, .css file in `dist/` folder. later we will use the `dist/` directory for the root of the each port in nginx config.

additional modification for hubs admin.

modify the reticulum.
find `lib/ret_web/router.ex` file

comment the `pipe_through`

```
scope "/api/postgrest" do
  #pipe_through([:secure_headers, :auth_required, :admin_required, :proxy_api])
  forward("/", RetWeb.Plugs.PostgrestProxy)
end
```

#### 6.2.4 Run postgREST server

Download postREST

```
sudo apt install libpq-dev
wget https://github.com/PostgREST/postgrest/releases/download/v9.0.0/postgrest-v9.0.0-linux-static-x64.tar.xz
tar -xf postgrest-v9.0.0-linux-static-x64.tar.xz
```

On reticulum iex

paste this

```
jwk = Application.get_env(:ret, Ret.PermsToken)[:perms_key] |> JOSE.JWK.from_pem(); JOSE.JWK.to_file("reticulum-jwk.json", jwk)
```

then it will create `reticulum-jwk.json` in your reticulum directory

Make `reticulum.conf` file

```
nano reticulum.conf
```

and paste

```
# reticulum.conf
db-uri = "postgres://postgres:postgres@localhost:5432/ret_dev"
db-schema = "ret0_admin"
db-anon-role = "postgres_anonymous"
jwt-secret = "@/absolute_path_to_your_file/reticulum-jwk.json"
jwt-aud = "ret_perms"
role-claim-key = ".postgrest_role"
```

Make new services using this command

```
sudo nano /etc/systemd/system/hubs-postgrest.service
```

and paste this

```
[Unit]
Description=Mozilla Hubs Postgrest Service

[Service]
ExecStart=/home/your_username/postgrest/postgrest reticulum.conf
User=your_username
WorkingDirectory=/home/your_username/postgrest

[Install]
WantedBy=multi-user.target
```

then start it with:

**Start**

```
sudo systemctl start hubs-postgrest
```

**Stop**

```
sudo systemctl stop hubs-postgrest
```

**Status**

```
sudo systemctl status hubs-postgrest
```

More about this is in [this](https://github.com/mozilla/hubs-ops/wiki/Running-PostgREST-locally)

#### 6.2.5 Make sure it runs well

Make sure the process name is same as in [.yml files](#node-js-based)

run

```
pm2 status
```

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

On `/home/admin/` dir, make .sh file named `start_reticulum_server.sh`

```bash
#!/bin/bash

echo "STARTING RETICULUM SERVER"
export PATH=$PATH:/home/admin/.asdf/shims
echo $PATH

cd /home/admin/hubs-actions-runner/reticulum/_work/reticulum/reticulum
(lsof -ti:4000) && kill -9 $(lsof -ti:4000)
MIX_ENV=prod mix release --overwrite
PORT=4000 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
sudo systemctl start hubs-postgrest

sleep 3

(lsof -ti:4000) && echo "Server started"
```

Info: the export path is for crontab knows the `mix` command location

Then make the .sh file is executable

```
chmod +x start_reticulum_server.sh
```

Open the crontab with this command. dont use sudo

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

We must pass everything to port 4000. So setting up `proxy_pass` on Nginx

And also open port for static file for hubs, hubs admin, spoke.

Open the Nginx config file with

```
sudo nano /etc/nginx/sites-available/default
```

And replace the content with this code

```
server {
        root /home/admin/hubs-actions-runner/hubs/_work/hubs/hubs/admin/dist;

        listen [::]:8989 ssl ipv6only=on;
        listen 8989 ssl;

        add_header Access-Control-Allow-Origin https://example.com;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
        root /home/admin/hubs-actions-runner/hubs/_work/hubs/hubs/dist;

        listen [::]:8080 ssl ipv6only=on;
        listen 8080 ssl;

        add_header Access-Control-Allow-Origin https://example.com;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
        root /home/admin/hubs-actions-runner/spoke/_work/spoke/spoke/dist;

        listen [::]:9090 ssl ipv6only=on;
        listen 9090 ssl;

        add_header Access-Control-Allow-Origin https://example.com;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name example.com;

    #hubs must not serve with http, so redirect it to https
    return 301 https://$host$request_uri;
}


server {
    server_name example.com;

     location / {
        #match everything
        rewrite ^\/(.*)$ /$1 break;
        # Proxy passing to port 4000
        proxy_pass https://example.com:4000;


        # The Important Websocket Bits!
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

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

# Next step

If everything is work next step is memory efficiency (RAM).
Less memory means less money you will spend.
The repo about efficiency i make it private. maybe later i will open patreon page.

Hubs which i install in my server is using 1.2 GB of RAM.

<br>
<br>

<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Tips for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)

<br>
<br>

# Useful reference

[Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)

[TCP vs UDP](https://www.youtube.com/watch?v=uwoD5YsGACg)
