## Install Resources Monitoring for VPS

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