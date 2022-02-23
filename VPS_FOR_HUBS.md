# Introduction

This Article is about hosting the mozilla hubs into vps / self hosted server. I spend 4 days of trying to install hubs on vps.

You must understand [installing mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) before you read this article.

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

# Installation

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


## 3. Setting up https for your domain

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

## 4. Install resources monitoring for vps

We are using [webmin](https://www.webmin.com/deb.html). this for monitoring server resources like CPU, RAM, and memory.
We dont know the resource usage hubs yet, so we must monitoring it.

Maybe on installation of webmin you got some error like me:

**Can't connet on port 10000**

The default port of webmin is 10000 but some ISP block that port. so we need to change with 1000. [see](https://serverfault.com/a/578397)

Don't forget, we must allow rules on `ufw` firewall

```
sudo ufw allow proto tcp from any to any port 1000
```

**HTTPS error**

If you can't open webmin ( port 1000) on chrome, please use mozilla firefox.

Go to [this web](https://www.inmotionhosting.com/support/product-guides/cloud-server/ssl-errors-and-https-in-webmin/)

Upload your certificates file that we have download before.

## 5. Make sure it on private repository

The entire hubs (reticulum,dialog,hubs,spoke) make it private repo. just to be sure it safe.

## 7. Install Dependencies

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


## 8. Setting up github actions

### 8.1 Elixir based

**Reticulum**

On your  reticulum repository goto actions tab on github. and create new workflow then choose elixir. 

it will create `.github/workflow/elixir.yml`

change the content `elixir.yml` with

```yml
name: Elixir CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

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
        elixir-version: '1.12.3' 
        # Define the OTP version [required]
        otp-version: '23.1' 
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


### 8.2 Node js based

Goto the action tab and new workflow -> choose node js

#### Hubs

Setting up yml file like this

```yml
name: Node.js CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

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
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

name: Node.js CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

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
        cache: 'npm'
        
    - run: yarn install
    - run: pm2 restart spoke_server
```

#### Dialog

Setting up yml file like this

```yml
name: Node.js CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

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

### 8.3 Add self hosted

Above you can see `runs-on: self-hosted` it means the command bellow it, will run on your server.

![Setting](/docs_img/self-hosted.png)

There you can follow the tutorial provided by github

I suggest you to setting clean folder

```
root
    hubs_project      <- wrap up with some folder
        hubs          <- where you put gihub action runner
        reticulum     <- where you put gihub action runner
        dialog        <- where you put gihub action runner
        spoke         <- where you put gihub action runner
```


## 9. Run all

### 9.1 Elixir based

On reticulum

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


### 9.2 Node js based



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
