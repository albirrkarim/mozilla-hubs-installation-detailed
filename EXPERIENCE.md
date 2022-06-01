# My experience 

Here I share about installing, and hardware resource used by mozilla hubs.

## 1. My own server

I have 16 GB of RAM

Intel(R) Xeon(R) Gold 6252 CPU @ 2.10GHz, 8 cores

and this program takes only 2.5 GB of RAM

if i run build command `npm run build` it will rise to about 4 GB.

## 2. Alibaba Cloud Elastic Compute Service

Im from indonesia and some guy in china ask me to install this hubs on their server (in china mainland). via ssh remote.

The hardware is 8cpu 32g ram 40gb disk

### - Download or Server Internet is Slow

You can use the VPN, i use nord VPN and select hongkong location (nearest). in this vpn not provide connection directly on mainland china. So i choose hongkong that near from china. then the server internet by magic is fast hahaha.

### - OS and Python version

Using ubuntu 18 (python 3.6) i got many problem. then i ask him to install ubuntu 20.

Ubuntu 20 comes with python 3.8 which is supporting the mediasoup v3 in dialog 

[requirement for mediasoup](https://mediasoup.org/documentation/v3/mediasoup/installation/)

### - Github empty response

When i install spoke with `yarn install` it says can't connect blabla...

then i run `ping github.com` and got 100% lost packet.

then i restart the server `sudo reboot` and still not work.

then i do [gnutls_handshake() failed GIT repository – AWS codecommit](https://devopscube.com/gnutls-handshake-failed-aws-codecommit/) and [How to resolve "git pull,fatal: unable to access 'https://github.com...\': Empty reply from server"](https://stackoverflow.com/questions/27087483/how-to-resolve-git-pull-fatal-unable-to-access-https-github-com-empty) and still not work

then i just `git clone the repo` like `three.js` and `gltf-webpack-loader` then i make it local library (on vps) with `yarn link`

and on `package.json` in spoke dir i remove that `three.js` and `gltf-webpack-loader` 

so i do 
```
yarn install
yarn link three
yarn link gltf-webpack-loader
```

<!-- ### - Yarn install error network connection.

info There appears to be trouble with your network connection. Retrying...

Increase the timeout

```
yarn config set network-timeout 6000000 -g
``` -->

### - Can't install erlang with asdf

I succeed install elixir with asdf but not erlang

Then i install erlang 23.3 with `.deb` file. visit this to [download](https://www.erlang-solutions.com/downloads/)

### - mix deps.get can't get dependecies from github

What i do is delete `.git` folder on `hubs-actions-runner/reticulum/_work/reticulum/reticulum`
then run `mix deps.get` again

### - Unlogic solve that i can't explain hahaha

Sometimes the server is unlogic. 

Problem A + solution A -> fail

Problem A + solution B -> fail

Problem A + solution C -> fail

Couple hour later or doing restart / or change my VPN location

Problem A + solution A -> suceess


<br>
<br>

I hope this can help you to installing mozilla hubs. You have other experience, please send me pull request. you can help more people.

<br>
<br>

[Paypal](https://paypal.me/AlbirrKarim)

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>


## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Tips for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)