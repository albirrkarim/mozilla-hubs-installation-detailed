# Introduction

My journey installing mozilla hubs, im new on project like this. so im confused of course. 4 days of figuring how this program work and finaly i can running mozilla hubs on my macbook air m1.
I want to share with you how to do.

This is about running mozilla hubs on locally. this is detailed version, step by step what i do.
Give me star on this repository for supporting me to always update this.

Before we start lets see the requirement.

# Requirement:

### Hardware:
- at least 8GB of RAM
- recomended using fast CPU

### Software

- Node js installed. when im install this hubs i use v16

### Knowledge
I assume you already know 

- Javascript
- React js
- Basic Webpack dev server
- Basic Elixir and phoenix
- Basic Web Socket


# Overview

![System Overview](/docs_img/System_Overview.jpeg)

The image above made with [figma](figma.com) you can read more description on [documentation](https://hubs.mozilla.com/docs/system-overview.html)

<br/>
<br/>

# Attention !

There is major step Cloning and Preparation -> Setting up HTTPS (SSL) -> Running

# 1. Cloning and preparation

## 1.1 Reticulum

[Its](https://github.com/mozilla/reticulum) a backend server that using elixir and phoenix. 


### 1.1.1 Install this

1.Elixir and Erlang (Elixir 1.12 and erlang version 23)**

You can installing those with follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Becareful about the version of elixir and erlang.

2.Ansible

You can use `pip` to install. take a look on this [tutorial](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip)

### 1.1.2 run this command

1. `mix deps.get`
2. `mix ecto.create`
    * If step 2 fails, you may need to change the password for the `postgres` role to match the password configured `dev.exs`.
    * From within the `psql` shell, enter `ALTER USER postgres WITH PASSWORD 'postgres';`
    * If you receive an error that the `ret_dev` database does not exist, (using psql again) enter `create database ret_dev;`
3. From the project directory `mkdir -p storage/dev`

## 1.2 Dialog

Using mediasoup RTC this will handling audio and video realtime communication. like camera stream, share screen.


Setting up secret key

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

![Mozilla Spoke](/docs_img/spoke.png)

```
git clone https://github.com/mozilla/Spoke.git
cd Spoke
yarn install
```

## 1.4 Hubs

In this [repo](https://github.com/mozilla/hubs) contains the hubs client and hubs admin (hubs/admin)


![System Overview](/docs_img/hubs_overview.jpeg)

Run this and it will start on `localhost:8080`

```
git clone https://github.com/mozilla/hubs.git
cd hubs
npm ci
```

## 1.5 Hubs Admin

from the hubs repo you can move to `hubs/admin` then run

```
npm install
```


# 2. Setting up HTTPS (SSL)

all the server must serve with https. so inside the reticulum directory you must generate certificate and key file 

run command `mix phx.gen.cert` it will generate key `selfsigned_key.pem` and certificate `selfsigned.pem`


rename `selfsigned_key.pem` to `key.pem`
rename `selfsigned.pem` to `cert.pem`

in reticulum directory move that two file into `priv/` folder

In Mac OS

Open the `cert.pem` on the tab system find that certificate. then click twice and change to always trust.

![Https mozilla hubs](/docs_img/cert.png)

Select the `cert.pem` and `key.pem` and copy it. next step we will distribute those two file into hubs, hubs admin, spoke, dialog, and reticulum. 

Oke first setting up in the reticulum.


## 2.1 Setting https for reticulum

then change the `config/dev.exs` setting path for the certificate and key file.

```
config :my_app, MyAppWeb.Endpoint,
  ...
  https: [
    port: 4001,
    cipher_suite: :strong,
    keyfile: "priv/key.pem",
    certfile: "priv/cert.pem"
  ]
```


## 2.2 Setting https for hubs

Paste that file into `hubs/certs`

We run hubs with `npm run local` right?
so add additional params on `package.json`

`--https --cert certs/cert.pem --key certs/key.pem`

Like this picture

![ssl hubs](/docs_img/ssl_hubs.png)


## 2.3 Setting https for hubs admin

Paste that file into `hubs/admin/certs` 

We run hubs with `npm run local` right?
so add additional params on `package.json`

`--https --cert certs/cert.pem --key certs/key.pem`

Like this picture

![ssl hubs admin](/docs_img/ssl_hubs_admin.png)


## 2.4 Setting https for spoke

Paste that file into `spoke/certs`

We run spoke with `yarn start` right ?
So change the `start` command

![ssl hubs admin](/docs_img/ssl_spoke.png)


With this

```
cross-env NODE_ENV=development BASE_ASSETS_PATH=https://localhost:9090/ webpack-dev-server --mode development --https --cert certs/cert.pem --key certs/key.pem
```


## 2.5 Setting https for dialog

Paste that file into `dialog/certs`

rename `cert.pem` to `fullchain.pem`

rename `key.pem` to `privkey.pem`

![ssl hubs dialog](/docs_img/ssl_dialog_1.png)


# 3. Runing

Open five terminals. for each reticulum, dialog, spoke, hubs, hubs admin.

![Running preparation](/docs_img/ss.png)

## 3.1 Run reticulum
with command
```bash
iex -S mix phx.server
```

## 3.2 Run dialog
with command
```bash
MEDIASOUP_LISTEN_IP=127.0.0.1 MEDIASOUP_ANNOUNCED_IP=127.0.0.1 npm start
```

## 3.3 run spoke
with command
```bash
yarn start
```

## 3.4 run hubs and hubs admin
each with command
```bash
npm run local
```

![Running success](/docs_img/success.png)

Urrraaa, Now you can access

with lock symbol (SSL secure)

Hubs 

[https://localhost:4000](https://localhost:4000)

Hubs admin

[https://localhost:4000/admin](https://localhost:4000/admin)

Spoke

[https://localhost:4000/spoke](https://localhost:4000/spoke)