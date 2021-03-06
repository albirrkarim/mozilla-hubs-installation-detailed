# The problem I faced and I already solved

## Remember! If you don't get a problem like me you don't have to follow this

The origin hubs repo is always updating and we are running on different server so it comes with different kind of error.

Sometimes the error I face doesn't necessarily mean you also face it.


## - 502 server communication error in hubs admin like this [issue](https://github.com/mozilla/hubs/issues/4970#issue-1087523703)

Ok, let's give try to solve this.

The problem is an API call to this route

`/api/ita/admin-info`

and other with slash ita `/ita`

if we check where is the code in the reticulum which handles that
open the `router.ex` ita API call the `RetWeb.Plugs.ItaProxy` if you in vs code you can command+click / ctrl+click for find the function is called.

```elixir

scope "/api/postgrest" do
   pipe_through([:secure_headers, :auth_required, :admin_required, :proxy_api])
   forward("/", RetWeb.Plugs.PostgrestProxy)
end

```

### What is port 3000?

Thanks to the contributor that give me a clue about the port 3000

its a [PostgREST](https://postgrest.org/en/stable/index.html)

Thanks to daniel, Give me info about [Running PostgREST locally](https://github.com/mozilla/hubs-ops/wiki/Running-PostgREST-locally)

![hubs admin panel work](/docs_img/admin_panel_work.png)

## - Architecture Kit Failed to Load and Import

This problem is related to number 1 above. We don't need `CORS_PROXY_SERVER` so set it with an empty string

![env spoke](/docs_img/env_spoke.png)

and make condition like this picture bellow

![env spoke](/docs_img/env_spoke_1.png)


## - Spoke Assets Thumbnail not Showing on Production

Edit Api.js

![env spoke](/docs_img/spoke_failed_3.png)

## - Spoke Console Error prefetch-src 

I got error like this

![Spoke console error](/docs_img/spoke_console_error.png)

Then add rule `prefetch-src` like this.

![Spoke console fix](/docs_img/spoke_console_fix.png)

## - Upload Assets too Large on Spoke Production

It because nginx

Open up the nginx configuration file

```
sudo nano /etc/nginx/nginx.conf
```

add this `client_max_body_size` to the http section

```
http {
    
    # other line...

    client_max_body_size 100M;
}  
```

Then restart nginx

```
sudo systemctl restart nginx
```

for more detail see [this article](https://www.tecmint.com/limit-file-upload-size-in-nginx/)


## - Error: listen EADDRNOTAVAIL: address not available

I install this project on server in china. alibaba elastic compute service

then i got that error when trying to run Spoke and hubs admin.


![address error](/docs_img/address_error.png)

just remove `process.env.HOST_IP ||` like picture above.

and leave `"0.0.0.0"`

thanks to [this](https://stackoverflow.com/questions/53955562/node-js-error-listen-eaddrnotavail-52-1122)


## - My Experience Installing On Alibaba Elastic Compute Service

[see](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)


## - Dialog Error Keep Restarting on pm2

```
this._mediasoupRouter._transports.size
```

![dialog restart](/docs_img/dialog_restart.png)


## - MediaSoupError: port bind failed due to address not available
Change the prod command in `package.json` 

it because `MEDIASOUP_LISTEN_IP`. set value with `0.0.0.0`

```
MEDIASOUP_LISTEN_IP=0.0.0.0 MEDIASOUP_ANNOUNCED_IP=123.xxx.xxx.xxx HTTPS_CERT_FULLCHAIN=/etc/letsencrypt/live/example.com/fullchain.pem HTTPS_CERT_PRIVKEY=/etc/letsencrypt/live/example.com/privkey.pem DOMAIN=example.com node index.js
```

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