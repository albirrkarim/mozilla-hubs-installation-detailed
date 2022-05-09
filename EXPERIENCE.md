## My experience installing on alibaba elastic compute service

Im from indonesia and some guy in china ask me to install this hubs on their server. via ssh remote

### - Download or server internet is slow

You can use the VPN, i use nord VPN and select hongkong location. in this vpn not provide connection directly on mainland china.
so i choose hongkong that near from china. then the server internet by magic is fast hahaha.

### - OS and Python version

Using ubuntu 18 (python 3.6) i got many problem. then i ask him to install ubuntu 20.

Ubuntu 20 comes with python 3.8 which is supporting the mediasoup v3 in dialog 

[requirement for mediasoup](https://mediasoup.org/documentation/v3/mediasoup/installation/)

### - Can't connect to github 
when i install spoke with `yarn install` it says can't connect blabla...
then i run `ping github.com` and got 100% lost packet.

then i restart the server

`sudo reboot`

and still not work.

then i do [https://devopscube.com/gnutls-handshake-failed-aws-codecommit/] and [this](https://stackoverflow.com/questions/27087483/how-to-resolve-git-pull-fatal-unable-to-access-https-github-com-empty)

and still not work

then i just `git clone the repo` like `three.js` and `gltf-webpack-loader` then i make it local library (on vps) with `yarn link`

and on `package.json` in spoke dir i remove that `three.js` and `gltf-webpack-loader` 

so i do 
```
yarn install
yarn link three
yarn link gltf-webpack-loader
```

### - Can't install erlang with asdf

i succeed install elixir with asdf but not erlang

then i install erlang 23.3 with `.deb` file


### - Error that i can't explain hahaha

Sometimes the server is unlogic. stay tune i will update this experience sharing. 