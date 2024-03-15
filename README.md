# Introduction

![this repo about](/docs_img/this_repo.png)

The end purpose of this repo is to install fully self-hosted hubs. See [the difference between this repo and the hubs-cloud](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/DIFFERENCE.md) first.

## News

- 15 Feb 2024, **Mozilla Hubs is End**, On February 13, 2024, Mozilla announced an organization-wide restructuring impacting a number of products housed within the Mozilla Corporation. Unfortunately, Hubs is one of the products impacted and it will be shut down later this year. Mozilla will not continue active development or maintenance of Hubs codebases and other community resources. [Read more](https://hubs.mozilla.com/labs/sunset)

- 16 oct 2023. Now mozilla hubs launching [Mozilla Hubs Community Edition](https://hubs.mozilla.com/labs/welcoming-community-edition/), its the way you can install self-hosted hubs, its official method. but i havn't trying yet. Take my repo as development reference and supportive techinal thing. Please Follow the official one.

- feb 2022 - oct 2023. maybe this repo is the only way you can install self-hosted hubs.

Let's help each other.

If you have advice or experience sharing about:

- Best practice
- Server hosting
- Server resource usages
- Tips

Please send me pull request or send me a message.

You can send me a message via [discord](https://discord.com/)

albirrkarim#8171

Thank you for all stars, contributions, and funding. So this repo can keep updated and provide benefits to the public.

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

# Installing Mozilla Hubs Locally

My journey installing Mozilla hubs, I'm new to a project like this. so I'm confused. After 4 days of figuring out how this program work finally I can run Mozilla hubs on my MacBook air m1.
I want to share with you how to do it.

For make it clear i provide this tutorial in [youtube video](https://youtu.be/KH1T9u9DaCo). give me like and subscribe to supporting me making the other video about this.

This is about running Mozilla hubs locally. this is a detailed version, step by step what I do.

Remember! if you got a problem with npm or dependency that you can't to solve for 1 hour. Just restart your PC. Trust me.

Also, I have done [Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

<br>
<br>

## Warning - This is not official documentation

<br>
<br>

## Disclaimer - Do with your own risk.

<br>
<br>

<br>
<br>

## This method still work, last time i checked 6 April 2023

<br>
<br>

# Requirement:

### Hardware:

- at least 8GB of RAM
- recommended using fast CPU

### Software

- Node js installed. when I install this hubs I use v16

### Knowledge

I assume you already know, if no you must up-skill first

![Up skill](/docs_img/excercise.gif)

- Javascript
- React js
- Basic Webpack dev server
- Basic Elixir and phoenix
- Basic Web Socket

# Overview

![System Overview](/docs_img/System_Overview.png)

The image above made with [figma](https://www.figma.com/) you can read more description on [documentation](https://hubs.mozilla.com/docs/system-overview.html)

I also try to make software overview, architecture, and tables on the database. you can see my [figma project](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

### Summary

Reticulum is the main host. it sync position, rotation, state of object. Comunicates with client browser through http request and websocket.

Dialog sync video and audio user. comunicates with clients browser through websocket.

Hubs, Spoke serve static assets then reticulum takes it and forward to client browser.

postREST is a server that help hubs Admin to doing basic task like CRUD (create read update delete)

Hubs Admin use websocket to comunicates with postgREST for authentication (login). for CRUD purpose hubs admin send http request (GET, POST, etc) to reticulum then reticulum doing proxy pass to postgREST.

<br/>
<br/>

# Attention!

There is major step [Cloning and Preparation](#1-cloning-and-preparation) -> [Setting up HOST](#2-setting-up-host) -> [Setting up HTTPS (SSL)](#3-setting-up-https-ssl) -> [Running](#4-running)

# 1. Cloning and preparation

## 1.1 Reticulum

It's a backend server that uses elixir and phoenix.

### 1.1.1 Clone

```bash
git clone https://github.com/mozilla/reticulum.git
cd reticulum
```

### 1.1.2 Install requirement

#### Postgres Database

Install on [linux ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)

Install on mac

With brew for installing CLI Postgres

```
brew install postgres
```

Then create user/change password

user: `postgres`

password : `postgres`

and alter it

```
ALTER USER postgres WITH SUPERUSER
```

#### Elixir and Erlang (Elixir 1.14.4 and erlang version 23.3)

You can install those with follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

Be careful about the version of elixir and erlang.

<!-- **Ansible**

You can use `pip` to install. take a look at this [tutorial](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip) -->

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

<!-- 4. Find on google how to install coturn, and manage it

[install coturn on ubuntu](https://ourcodeworld.com/articles/read/1175/how-to-create-and-configure-your-own-stun-turn-server-with-coturn-in-ubuntu-18-04)

5. Edit the Dialog [configuration file](https://ourcodeworld.com/articles/read/1175/how-to-create-and-configure-your-own-stun-turn-server-with-coturn-in-ubuntu-18-04) `turnserver.conf` and update the PostgreSQL database connection string to use the _coturn_ schema from the Reticulum database:

```
psql-userdb="host=localhost dbname=ret_dev user=postgres password=postgres options='-c search_path=coturn' connect_timeout=30"
``` -->

## 1.2 Dialog

Using mediasoup RTC will handle audio and video real-time communication. like camera stream, share screen.

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

In here you can create/edit the scenes/buildings whatever you call it.

### 1.3.1 Clone

![Mozilla Spoke](/docs_img/spoke.png)

```
git clone https://github.com/mozilla/Spoke.git
cd Spoke
yarn install
```

### 1.3.2 Set the Host (optional)

on the `package.json` look at the `run-local-reticulum` script change the host. `hub.local` or `localhost`.

in this tutorial i use `localhost`;

## 1.4 Hubs

In this [repo](https://github.com/mozilla/hubs) contains the hubs client and hubs admin (hubs/admin)

![System Overview](/docs_img/hubs_overview.jpeg)

Clone and install dependencies

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

All the servers must serve with HTTPS. you must generate a certificate and key file

## 3.1 Generating certificate and making it trust

Open terminal in reticulum directory

run command

```bash
mix phx.gen.cert
```

It will generate key `selfsigned_key.pem` and certificate `selfsigned.pem` in the `priv/cert` folder

Rename `selfsigned_key.pem` to `key.pem`

Rename `selfsigned.pem` to `cert.pem`

#### Now we have `key.pem` and `cert.pem` file

In Mac OS, I don't know in windows or Linux. please find it yourself

Open the `cert.pem` on the tab system find that certificate then clicks twice and change to always trust.

![Https mozilla hubs](/docs_img/cert.png)

Select the `cert.pem` and `key.pem` and copy it. next step we will distribute those two files into hubs, hubs admin, spoke, dialog, and reticulum.

Oke first set up in the reticulum.

## 3.2 Setting https for reticulum

On the `config/dev.exs` We must be setting the path for the certificate and key file.

![Https mozilla hubs](/docs_img/cert_1.png)

## 3.3 Setting HTTPS for hubs

Paste [that](#now-we-have-keypem-and-certpem-file) file into `hubs/certs`

## 3.4 Setting HTTPS for hubs admin

Paste [that](#now-we-have-keypem-and-certpem-file) file into `hubs/admin/certs`

## 3.5 Setting HTTPS for spoke

Paste [that](#now-we-have-keypem-and-certpem-file) file into `spoke/certs`

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

`127.0.0.1` is the default IP of localhost on Mac / Linux you can look at the IP with this command:

```bash
sudo nano /etc/hosts
```

## 4.3 Run spoke

with command

```bash
yarn run-local-reticulum
```

## 4.4 Run hubs

Each with command

```bash
npm run local
```

## 4.5 Run postgREST server

More about this is in [this](https://github.com/mozilla/hubs-ops/wiki/Running-PostgREST-locally)

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

then the folder looks like this (contain two files)

```
/
   postgrest
   reticulum.conf
```

then run postREST with

```
postgrest reticulum.conf
```

## 4.6 Run Hubs Admin

This is tricky.

Goto reticulum folder then find `router.ex`, then find `/api/postgrest` and modify to this code bellow.

```elixir
scope "/api/postgrest" do
   # pipe_through [:secure_headers, :auth_required, :admin_required, :proxy_api]
   if(Mix.env() == :prod) do
   pipe_through([:secure_headers])
   end

   forward "/", RetWeb.Plugs.PostgrestProxy
end
```

The first way is we can use. `npm run local` but if we use this as a live webpack server.

IF, This tab doesn't work. (pending scenes,etc)

![](./docs_img/build_admin.png)

The solution is we need to run it as a static file using nginx.

make new file `.prod.env`

```
# To override these variables, create a .env file containing the overrides.
CONFIGURABLE_SERVICES="janus-gateway,reticulum,hubs,spoke"

ITA_SERVER="https://localhost:3333"
RETICULUM_SERVER="localhost:4000"

# PostgREST server configured to allow administrative access to the db.
POSTGREST_SERVER="https://localhost:4000/api/postgrest"
BASE_ASSETS_PATH="https://localhost:8989/"
```

then modify the `webpack.config.js`

find the code that talk about `.env`

the modify like this:

```js
if (argv.mode === "production") {
  dotenv.config({ path: ".prod.env" });
} else {
  dotenv.config({ path: ".env" });
  dotenv.config({ path: ".defaults.env" });
}
```

build it first `npm run build` then you can see there's a `dist` folder inside the `hubs/admin`

I use mac book, in the macbook just install `nginx` then

find the nginx config file

`sudo nginx -t`

OK, let's assume you have knowledge about nginx before.

```nginx
server {
      listen 8989 ssl;
      server_name localhost;

      location / {
         root   /absolute_path/hubs/admin/dist;
         index  admin.html index.htm;
      }

      ssl_certificate /absolute_path/hubs/admin/certs/cert.pem;
      ssl_certificate_key /absolute_path/hubs/admin/certs/key.pem;
}
```

Don't forget to restart nginx

then access the hubs admin via reticulum `https://localhost:4000/admin`

<br>

Now you can access, with lock symbol (SSL secure)

| Server     | URL                                                          |
| ---------- | ------------------------------------------------------------ |
| Hubs       | [https://localhost:4000](https://localhost:4000)             |
| Hubs admin | [https://localhost:4000/admin](https://localhost:4000/admin) |
| Spoke      | [https://localhost:4000/spoke](https://localhost:4000/spoke) |

<br>
<br>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

## Also read:

[How to Maintenance Server (Backup, etc)](MAINTENANCE.md)

[Hosting Mozilla Hubs on VPS](VPS_FOR_HUBS.md)

[The Problem I Still Faced](PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](PROBLEM_SOLVED.md)

[Tips for Modification](HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](EXPERIENCE.md)