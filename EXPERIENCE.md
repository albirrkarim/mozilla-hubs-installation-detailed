# My experience 

Here I share about installing, and hardware resource used by mozilla hubs.

## 1. Alibaba Cloud Elastic Compute Service

Im from indonesia and someone in china ask me to install this hubs on their server (in china mainland). via ssh remote.

The hardware is 8cpu 32g ram 40gb disk

### - Download or Server Internet is Slow

You can use the VPN, i use nord VPN and select hongkong location (nearest). in this vpn not provide connection directly on mainland china. So i choose hongkong that near from china. then the server internet by magic is fast.

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

### - Can't install erlang with asdf

I succeed install elixir with asdf but not erlang

Then i install erlang 23.3 with `.deb` file. visit this to [download](https://www.erlang-solutions.com/downloads/)

### - mix deps.get can't get dependecies from github

What i do is delete `.git` folder on `hubs-actions-runner/reticulum/_work/reticulum/reticulum`
then run `mix deps.get` again

### - Unlogic solve that i can't explain

Sometimes the server is unlogic. 

Problem A + solution A -> fail

Problem A + solution B -> fail

Couple hour later or doing restart / or change my VPN location

Problem A + solution A -> suceess


### But all error above maybe can be solve by chatting with customer service

Here the answer:

Hello, check that your server is a machine in Shenzhen, the github you are visiting is an overseas site, and the mainland server accesses overseas sites through international links and operator routing nodes, which will be subject to international link congestion, and operators Outbound routing restrictions are prone to network link congestion, unstable access, and excessive access delay.

Check that your server is using the EIP, try changing the EIP and try again. 

<br>
<br>

I hope this can help you to installing mozilla hubs. You have other experience, please send me pull request. you can help more people.

<br>
<br>

<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>


## Also read:

[List Hubs Custom Features](https://github.com/albirrkarim/mozilla-hubs-custom-features)

[Hubs Memory Efficiency & Usage Simulation (Private Repo)](https://github.com/albirrkarim/mozilla-hubs-optimization)

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Tips for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)