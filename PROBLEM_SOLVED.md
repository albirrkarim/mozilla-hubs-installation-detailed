# The problem I faced and I already solved

## If you don't get a problem like me you don't have to follow this


## 1. Architecture Kit Failed to load and import

This problem is related to number 1 above. We don't need `CORS_PROXY_SERVER` so set it with an empty string

![env spoke](/docs_img/env_spoke.png)

and make condition like this picture bellow

![env spoke](/docs_img/env_spoke_1.png)


## 2. Spoke assets thumbnail not showing on production

Edit Api.js

![env spoke](/docs_img/spoke_failed_3.png)

## 3. Spoke console error prefetch-src 

I got error like this

![Spoke console error](/docs_img/spoke_console_error.png)

Then add rule `prefetch-src` like this.

![Spoke console fix](/docs_img/spoke_console_fix.png)

## 3. Upload assets too large on spoke production

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

<br>
<br>

## IF you have questions feel free to open an issue

<br>
<br>

## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Basic Information for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)