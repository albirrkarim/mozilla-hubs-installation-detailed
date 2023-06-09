I will share my experience when i try to install this on Cloud Server.

Specifically on alibaba ECS

## Assets transfer will use local and international bandwidth

The Cloud server has local bandwidth and international bandwidth.

![System Overview Production Server](/docs_img/Production_Overview_Sample_1_Basic_new.png)

with that architecture using nginx and refering to this [nginx config](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md#7-setting-up-nginx)

```nginx
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

    #other config ...
}
```

The local bandwidth is fast, but you must pay more for international bandwidth so it will fast.

And that nginx config says that any request from the internet will forwarded to `example.com:4000`. it mean it will send using international bandwidth.

![System Overview Production Server](/docs_img/nginx_problem.jpeg)


So what the solution for it ? 

using IP tables.

But when i try to using IP tables theres other problem again with the SSL.

Then i stop to solve this issue for a moment because it will takes long time and now i still working on my thesis for my master degrees.