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


<br>
<br>

## IF you have a questions feel free to open an issue


<br>
<br>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>


## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The problem i still faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The problem i faced and i already solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Basic information for modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)