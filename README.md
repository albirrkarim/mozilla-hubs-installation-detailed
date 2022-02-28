# Introduction

My journey installing mozilla hubs, im new on project like this. so im confused of course. 4 days of figuring how this program work and finaly i can running mozilla hubs on my macbook air m1.
I want to share with you how to do.

This is about running mozilla hubs on locally. this is detailed version, step by step what i do.


I have done [Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

Now, i try to make backend services for hubs admin

Give me star on this repository or giving a donation with this button bellow for supporting me to always update this.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Q5Q0BC92X)

<br>
<br>

## Disclaimer - this tutorial maybe isn't a best practice

<br>
<br>

# Requirement:

### Hardware:

- at least 8GB of RAM
- recomended using fast CPU

### Software

- Node js installed. when im install this hubs i use v16

### Knowledge

I assume you already know, if no you must up skill first

![Up skill](/docs_img/excercise.gif)

- Javascript
- React js
- Basic Webpack dev server
- Basic Elixir and phoenix
- Basic Web Socket

# Overview

![System Overview](/docs_img/System_Overview.jpeg)

The image above made with [figma](https://www.figma.com/) you can read more description on [documentation](https://hubs.mozilla.com/docs/system-overview.html)

I try to make sofware overview, architecure, and tables on database. you can see my [figma project](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0)

<br/>
<br/>

# Attention !

There is major step [Cloning and Preparation](#1-cloning-and-preparation) -> [Setting up HOST](#2-setting-up-host) -> [Setting up HTTPS (SSL)](#3-setting-up-https-ssl) -> [Running](#4-runing)

# 1. Cloning and preparation

## 1.1 Reticulum

Its a backend server that using elixir and phoenix.

### 1.1.1 Clone

```bash
git clone https://github.com/mozilla/reticulum.git
cd reticulum
```

### 1.1.2 Install requirement

**Postgres Database**

https://postgresapp.com/downloads.html

**Elixir and Erlang (Elixir 1.12 and erlang version 23)**

You can installing those with follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Becareful about the version of elixir and erlang.

**Ansible**

You can use `pip` to install. take a look on this [tutorial](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip)

### 1.1.3 run this command

1. `mix deps.get`
2. `mix ecto.create`
   - If step 2 fails, you may need to change the password for the `postgres` role to match the password configured `dev.exs`.
   - From within the `psql` shell, enter `ALTER USER postgres WITH PASSWORD 'postgres';`
   - If you receive an error that the `ret_dev` database does not exist, (using psql again) enter `create database ret_dev;`
3. From the project directory `mkdir -p storage/dev`

### 1.1.4 Run Reticulum against a local Dialog instance

1. Update the Janus host in `dev.exs`:

```elixir
dev_janus_host = "localhost"
```

2. Update the Janus port in `dev.exs`:

```elixir
config :ret, Ret.JanusLoadStatus, default_janus_host: dev_janus_host, janus_port: 4443
```

3. Add the Dialog meta endpoint to the CSP rules in `add_csp.ex`:

```elixir
default_janus_csp_rule =
   if default_janus_host,
      do: "wss://#{default_janus_host}:#{janus_port} https://#{default_janus_host}:#{janus_port} https://#{default_janus_host}:#{janus_port}/meta",
      else: ""
```

4. Find on google how to install coturn, and manage it

[install coturn on ubuntu](https://ourcodeworld.com/articles/read/1175/how-to-create-and-configure-your-own-stun-turn-server-with-coturn-in-ubuntu-18-04)

5. Edit the Dialog [configuration file](https://ourcodeworld.com/articles/read/1175/how-to-create-and-configure-your-own-stun-turn-server-with-coturn-in-ubuntu-18-04) `turnserver.conf` and update the PostgreSQL database connection string to use the _coturn_ schema from the Reticulum database:

```
psql-userdb="host=localhost dbname=ret_dev user=postgres password=postgres options='-c search_path=coturn' connect_timeout=30"
```

## 1.2 Dialog

Using mediasoup RTC this will handling audio and video realtime communication. like camera stream, share screen.

### 1.2.1 Clone and get dependencies

```bash
git clone https://github.com/mozilla/dialog.git
cd dialog
npm install
```

### 1.2.2 Setting up secret key

thanks to this [comment](https://github.com/mozilla/hubs/discussions/3323#discussioncomment-1857495)

Generate RSA (Public and Private key) with [generator online](https://travistidwell.com/jsencrypt/demo/)

make empty file `perms.pub.pem` and fill it with RSA Public key

![RSA generator ](/docs_img/rsa.png)
![Paste](/docs_img/rsa_1.png)

Goto reticulum directory on `reticulum/config/dev.exs` change PermsToken with the RSA private key that you generate before.

```elixir
config :ret, Ret.PermsToken, perms_key: "-----BEGIN RSA PRIVATE KEY----- paste here copyed key but add every line \n before END RSA PRIVATE KEY-----"
```

## 1.3 Spoke

In here you can create / edit the scenes / buildings whatever you call it.

### 1.3.1 Clone

![Mozilla Spoke](/docs_img/spoke.png)

```
git clone https://github.com/mozilla/Spoke.git
cd Spoke
yarn install
```

### 1.3.2 Set the base routes

I hope you know the basic `react-router-dom` with the defaut url in slash `/` on `localhost:9090`

But in the end we will access the spoke on `localhost:4000/spoke`

So we must set base URL to `/spoke`

Add the `ROUTER_BASE_PATH=/spoke` params to the `start` command on `package.json`


![Mozilla Spoke](/docs_img/spoke_change.png)

```
cross-env NODE_ENV=development ROUTER_BASE_PATH=/spoke BASE_ASSETS_PATH=https://localhost:9090/ webpack-dev-server --mode development --https --cert certs/cert.pem --key certs/key.pem
```

## 1.4 Hubs

In this [repo](https://github.com/mozilla/hubs) contains the hubs client and hubs admin (hubs/admin)

![System Overview](/docs_img/hubs_overview.jpeg)

Clone and installing dependencies

```
git clone https://github.com/mozilla/hubs.git
cd hubs
npm ci
```

## 1.5 Hubs Admin

from the [hubs repo](#14-hubs) you can move to `hubs/admin` then run

```
npm install
```

# 2. Setting up HOST

We are not using `hubs.local` domain. we use `localhost`

so change every host configuration on reticulum, dialog, hubs, hubs admin, spoke.

# 3. Setting up HTTPS (SSL)

All the server must serve with https. you must generate certificate and key file

## 3.1 Generating Certificate and make it trust

Open terimnal in reticulum directory

run command

```bash
mix phx.gen.cert
```

It will generate key `selfsigned_key.pem` and certificate `selfsigned.pem` in the `piv/cert` folder

Rename `selfsigned_key.pem` to `key.pem`

Rename `selfsigned.pem` to `cert.pem`

#### Now we have `key.pem` and `cert.pem` file

In Mac OS, I don't know in windows or linux. please find it your self

Open the `cert.pem` on the tab system find that certificate then click twice and change to always trust.

![Https mozilla hubs](/docs_img/cert.png)

Select the `cert.pem` and `key.pem` and copy it. next step we will distribute those two file into hubs, hubs admin, spoke, dialog, and reticulum.

Oke first setting up in the reticulum.

## 3.2 Setting https for reticulum

On the `config/dev.exs` We must setting path for the certificate and key file.

![Https mozilla hubs](/docs_img/cert_1.png)

## 3.3 Setting https for hubs

Paste [that](#now-we-have-keypem-and-certpem-file) file into `hubs/certs`

We run hubs with `npm run local` right?
so add additional params on `package.json`

`--https --cert certs/cert.pem --key certs/key.pem`

Like this picture

![ssl hubs](/docs_img/ssl_hubs.png)

## 3.4 Setting https for hubs admin

Paste [that](#now-we-have-keypem-and-certpem-file) file into `hubs/admin/certs`

We run hubs with `npm run local` right?
so add additional params on `package.json`

`--https --cert certs/cert.pem --key certs/key.pem`

Like this picture

![ssl hubs admin](/docs_img/ssl_hubs_admin.png)

## 3.5 Setting https for spoke

Paste [that](#now-we-have-keypem-and-certpem-file) file into `spoke/certs`

We run spoke with `yarn start` right ?
So change the `start` command

![ssl hubs admin](/docs_img/ssl_spoke.png)

With this

```
cross-env NODE_ENV=development ROUTER_BASE_PATH=/spoke BASE_ASSETS_PATH=https://localhost:9090/ webpack-dev-server --mode development --https --cert certs/cert.pem --key certs/key.pem
```

Short description:

BASE_ASSETS_PATH = basicaly we run the spoke on localhost:9090

## 3.6 Setting https for dialog

Paste [that](#now-we-have-keypem-and-certpem-file) file into `dialog/certs`

rename `cert.pem` to `fullchain.pem`

rename `key.pem` to `privkey.pem`

![ssl hubs dialog](/docs_img/ssl_dialog_1.png)

# 4. Running

Open five terminals. for each reticulum, dialog, spoke, hubs, hubs admin.

![Running preparation](/docs_img/ss.png)

## 4.1 Run reticulum

with command

```bash
iex -S mix phx.server
```

## 4.2 Run dialog

Edit the `start` command on the package.json with 

```
MEDIASOUP_LISTEN_IP=127.0.0.1 MEDIASOUP_ANNOUNCED_IP=127.0.0.1 DEBUG=${DEBUG:='*mediasoup* *INFO* *WARN* *ERROR*'} INTERACTIVE=${INTERACTIVE:='true'} node index.js
```

For giving params `MEDIASOUP_LISTEN_IP` and `MEDIASOUP_ANNOUNCED_IP`

![Running preparation](/docs_img/run_dialog.png)

Start dialog server with command:
```
npm run start
```

`127.0.0.1` is default IP of localhost on Mac / Linux you can look the IP with this command:

```bash
sudo nano /etc/hosts
```

## 4.3 Run spoke

with command

```bash
yarn start
```

## 4.4 Run hubs and hubs admin

each with command

```bash
npm run local
```

Urrraaa, Now you can access

with lock symbol (SSL secure)

Hubs

[https://localhost:4000](https://localhost:4000)

Hubs admin

[https://localhost:4000/admin](https://localhost:4000/admin)

Spoke

[https://localhost:4000/spoke](https://localhost:4000/spoke)

<br>
<br>

## IF you have a questions feel free to open an issue


<br>
<br>

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Q5Q0BC92X)


## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The problem i still faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The problem i faced and i already solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Basic information for modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)