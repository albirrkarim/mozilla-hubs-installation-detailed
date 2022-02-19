# Introduction

This Article is about hosting the mozilla hubs into vps / self hosted server


# Requirement

Knowledge
- Basic CLI linux
- Github Actions, if you dont know please goto youtube.
- Remoting VPS via ssh

Software
- VPS with ubuntu based

# Installation


## 1. First step
Login with SSH
```
ssh username@your_IP
```

Update all to the latest
```
sudo su
apt-get update
apt-get upgrade
```

## 2. Install Firewall

we are using ufw. firewall is some software to blocking / allow request.

Please look at this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

## 3. Install resources monitoring for vps 
we are using [webmin](https://www.webmin.com/deb.html). this for monitoring server resources like CPU, RAM, and memory. 


## 4. Make sure it on private repository

The entire hubs (reticulum,dialog,hubs,spoke) make it private repo. just to be sure it safe.

## 5. 

wait I will update this soon. and give me star.


# Useful reference

[Automatic Deployment With Github Actions](https://www.youtube.com/watch?v=X3F3El_yvFg)